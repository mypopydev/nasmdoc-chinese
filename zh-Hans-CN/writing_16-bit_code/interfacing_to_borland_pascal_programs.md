7.5 Interfacing to Borland Pascal Programs
=====

       Interfacing to Borland Pascal programs is similar in concept to
       interfacing to 16-bit C programs. The differences are:

       (*) The leading underscore required for interfacing to C programs is
           not required for Pascal.

       (*) The memory model is always large: functions are far, data
           pointers are far, and no data item can be more than 64K long.
           (Actually, some functions are near, but only those functions
           that are local to a Pascal unit and never called from outside
           it. All assembly functions that Pascal calls, and all Pascal
           functions that assembly routines are able to call, are far.)
           However, all static data declared in a Pascal program goes into
           the default data segment, which is the one whose segment address
           will be in `DS' when control is passed to your assembly code.
           The only things that do not live in the default data segment are
           local variables (they live in the stack segment) and dynamically
           allocated variables. All data _pointers_, however, are far.

       (*) The function calling convention is different - described below.

       (*) Some data types, such as strings, are stored differently.

       (*) There are restrictions on the segment names you are allowed to
           use - Borland Pascal will ignore code or data declared in a
           segment it doesn't like the name of. The restrictions are
           described below.

## 7.5.1 The Pascal Calling Convention

       The 16-bit Pascal calling convention is as follows. In the following
       description, the words _caller_ and _callee_ are used to denote the
       function doing the calling and the function which gets called.

       (*) The caller pushes the function's parameters on the stack, one
           after another, in normal order (left to right, so that the first
           argument specified to the function is pushed first).

       (*) The caller then executes a far `CALL' instruction to pass
           control to the callee.

       (*) The callee receives control, and typically (although this is not
           actually necessary, in functions which do not need to access
           their parameters) starts by saving the value of `SP' in `BP' so
           as to be able to use `BP' as a base pointer to find its
           parameters on the stack. However, the caller was probably doing
           this too, so part of the calling convention states that `BP'
           must be preserved by any function. Hence the callee, if it is
           going to set up `BP' as a frame pointer, must push the previous
           value first.

       (*) The callee may then access its parameters relative to `BP'. The
           word at `[BP]' holds the previous value of `BP' as it was
           pushed. The next word, at `[BP+2]', holds the offset part of the
           return address, and the next one at `[BP+4]' the segment part.
           The parameters begin at `[BP+6]'. The rightmost parameter of the
           function, since it was pushed last, is accessible at this offset
           from `BP'; the others follow, at successively greater offsets.

       (*) The callee may also wish to decrease `SP' further, so as to
           allocate space on the stack for local variables, which will then
           be accessible at negative offsets from `BP'.

       (*) The callee, if it wishes to return a value to the caller, should
           leave the value in `AL', `AX' or `DX:AX' depending on the size
           of the value. Floating-point results are returned in `ST0'.
           Results of type `Real' (Borland's own custom floating-point data
           type, not handled directly by the FPU) are returned in
           `DX:BX:AX'. To return a result of type `String', the caller
           pushes a pointer to a temporary string before pushing the
           parameters, and the callee places the returned string value at
           that location. The pointer is not a parameter, and should not be
           removed from the stack by the `RETF' instruction.

       (*) Once the callee has finished processing, it restores `SP' from
           `BP' if it had allocated local stack space, then pops the
           previous value of `BP', and returns via `RETF'. It uses the form
           of `RETF' with an immediate parameter, giving the number of
           bytes taken up by the parameters on the stack. This causes the
           parameters to be removed from the stack as a side effect of the
           return instruction.

       (*) When the caller regains control from the callee, the function
           parameters have already been removed from the stack, so it needs
           to do nothing further.

       Thus, you would define a function in Pascal style, taking two
       `Integer'-type parameters, in the following way:

       global  myfunc 
       
       myfunc: push    bp 
               mov     bp,sp 
               sub     sp,0x40         ; 64 bytes of local stack space 
               mov     bx,[bp+8]       ; first parameter to function 
               mov     bx,[bp+6]       ; second parameter to function 
       
               ; some more code 
       
               mov     sp,bp           ; undo "sub sp,0x40" above 
               pop     bp 
               retf    4               ; total size of params is 4

       At the other end of the process, to call a Pascal function from your
       assembly code, you would do something like this:

       extern  SomeFunc 
       
              ; and then, further down... 
       
              push   word seg mystring   ; Now push the segment, and... 
              push   word mystring       ; ... offset of "mystring" 
              push   word [myint]        ; one of my variables 
              call   far SomeFunc

       This is equivalent to the Pascal code

       procedure SomeFunc(String: PChar; Int: Integer); 
           SomeFunc(@mystring, myint);

## 7.5.2 Borland Pascal Segment Name Restrictions

       Since Borland Pascal's internal unit file format is completely
       different from `OBJ', it only makes a very sketchy job of actually
       reading and understanding the various information contained in a
       real `OBJ' file when it links that in. Therefore an object file
       intended to be linked to a Pascal program must obey a number of
       restrictions:

       (*) Procedures and functions must be in a segment whose name is
           either `CODE', `CSEG', or something ending in `_TEXT'.

       (*) Initialised data must be in a segment whose name is either
           `CONST' or something ending in `_DATA'.

       (*) Uninitialised data must be in a segment whose name is either
           `DATA', `DSEG', or something ending in `_BSS'.

       (*) Any other segments in the object file are completely ignored.
           `GROUP' directives and segment attributes are also ignored.

## 7.5.3 Using `c16.mac' With Pascal Programs

       The `c16.mac' macro package, described in section 7.4.5, can also be
       used to simplify writing functions to be called from Pascal
       programs, if you code `%define PASCAL'. This definition ensures that
       functions are far (it implies `FARCODE'), and also causes procedure
       return instructions to be generated with an operand.

       Defining `PASCAL' does not change the code which calculates the
       argument offsets; you must declare your function's arguments in
       reverse order. For example:

       %define PASCAL 
       
       proc    _pascalproc 
       
       %$j     arg 4 
       %$i     arg 
               mov     ax,[bp + %$i] 
               mov     bx,[bp + %$j] 
               mov     es,[bp + %$j + 2] 
               add     ax,[bx] 
       
       endproc

       This defines the same routine, conceptually, as the example in
       section 7.4.5: it defines a function taking two arguments, an
       integer and a pointer to an integer, which returns the sum of the
       integer and the contents of the pointer. The only difference between
       this code and the large-model C version is that `PASCAL' is defined
       instead of `FARCODE', and that the arguments are declared in reverse
       order.

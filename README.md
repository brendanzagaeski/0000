## Overview

Discussions at work about the Mono interpreter, JIT compiler, and static
compiler made me wonder why interpreted programs are necessarily less efficient
than compiled programs.  I had also wanted to try writing something in
[colorForth][] or another small Forth-like language, so I thought a fun way to
get some hands-on insights about compilers and interpreters would be to write my
own for a Forth-like language.

  * [*fc.html*][fc] contains two implementations of a parser and compiler for a
    small Forth-like language.  One is written in the Forth-like language
    itself.  The other is written in JavaScript to bootstrap the language.

  * [*fi.html*][fi] contains corresponding interpreter implementations that
    reuse the parser logic.

As reading material, these implementations are probably more comical than
educational, but I would encourage anyone who wants to bootstrap a small
programming language or learn about CPU instructions to try a similar project of
your own.

If you're curious to see the `diff` between the compiler and interpreter, you
can compare *fc.html* and *fi.html* locally or look at the [second commit in
this repository][second-commit].  Note that the `diff` by itself might not
actually be very interesting.  Part of the problem is that it's hard to see from
the `diff` that the compiler can produce a stand-alone executable that is only a
few Intel 64 instructions long, whereas the interpreter needs a copy of *all the
interpreter instructions* to run a program.

## Comical things

For the sake of keeping the number of features to a minimum, the Forth-like
language only provides jumps to absolute memory addresses.  It does not provide
relative or labeled jumps.  To make that limitation more manageable, the
programs follow a rather hilarious coding style where every source line is
exactly 16 bytes long and compiles to a set of Intel 64 instructions that is
also exactly 16 bytes long.  That way, cursor positions in the source match up
with byte positions in the compiled executable, so it's relatively easy to
figure out jump addresses by hand.

The compilers output Intel 64 instructions directly instead of using another
assembler or compiler.  On the one hand, this makes sense because the
implementation of each Forth-like word needs to fit into exactly 16 bytes, and
using an intermediate assembler or compiler would make it harder to keep track
of the total instruction size of each word.  On the other hand, the Intel
instruction set is fairly complex, and I'm not an expert on it, so some of the
instruction sequences probably violate best practices of security, efficiency,
or correctness.  In the end, I didn't worry about that because this was just an
educational exercise.  Plus, hand-picking the instructions was fun.  I got to
learn about the daunting variety of available Intel 64 instruction encodings and
try to limit how many I used.

The JavaScript in these files uses several funny low-level idioms that imitate
the Forth-like code.  For example, it uses explicit loops and a stack to parse
the input four bytes at a time instead of calling `String.split ()` to tokenize
it all at once, and it implements its own hexadecimal number parsing instead of
calling `Number.parseInt ()`.  On top of that, it uses `String`s as the only
aggregate data type, even for the stack.  Since `String`s are immutable, that
means *every* stack operation allocates a new `String`, which gets quite
expensive if the stack is large.  Fortunately, the current programs are small
enough that this isn't a problem.  In hindsight, these implementation choices do
seem a little silly compared to "normal" JavaScript, but I had fun writing the
code this way, so I'd probably make the same silly choices again if I had it to
do over.

## How to run the compilers in *fc.html*

 1. Save *fc.html* locally.

 2. Navigate to the local copy of *fc.html* in a browser on any operating
    system.

 3. When prompted, save the compiled output to a local file named *fc0*.

 4. If you completed steps 2 and 3 on an operating system other than x86\_64
    Linux, move the file *fc0* to an x86\_64 Linux system.

 5. Set the file to be executable:

    ```sh
    chmod +x fc0
    ```

 6. Execute the program and direct the output into a new file *fc1*:

    ```sh
    ./fc0 > fc1
    ```

 7. Repeat steps 5 and 6 with *fc1* to create *fc2*, and then once more with
    *fc2* to create *fc3*:

    ```sh
    chmod +x fc1
    ./fc1 > fc2
    chmod +x fc2
    ./fc2 > fc3
    ```

 8. Set *fc3* to be executable, and then run it *without* redirecting the
    output:

    ```sh
    chmod +x fc3
    ./fc3
    ```

### Result

```
Hello, world!
```

### Notes

*fc.html* uses the compiler written in JavaScript to compile the compiler
written in the Forth-like language to a Linux x86\_64 ELF executable *fc0*.  To
test that *fc0* works correctly, *fc0* compiles its own source code to produce
*fc1*.  To test that *fc1* also works correctly, *fc1* compiles the compiler
written in the Forth-like language once more to produce *fc2*.  *fc2* then
compiles a final small "Hello, world!" program to produce *fc3*.

## How to run the interpreters in *fi.html*

 1. Save *fi.html* locally.

 2. Navigate to the local copy of *fi.html* in a browser on any operating
    system.

 3. When prompted, save the result to a local file and view the contents in a
    text viewer.

### Result

```
Hello, world!
```

## Future possibilities

One thing that bugs me about the current compiler and interpreter is that they
spend quite a few lines on *parsing*.  That makes it harder to see which parts
are essential for *compiling and interpreting*.  I'm tempted to modify the
Forth-like source format to store numbers directly as 31-bit little-endian
values and use the 32nd bit as a flag to distinguish between words and numbers.
That would let me cut out almost all of the lines related to parsing hexadecimal
numbers and whitespace.

It could be educational and entertaining to implement a limited example of
just-in-time compilation in this little system.

The fact that the Forth-like language doesn't provide a way to define new words
means I didn't get to try the nice Forth approach of writing a program as a
collection of small words.  I'll have to revisit that in the future.

## Inspirations

  * Discussions at work about the Mono [interpreter][mono-interpreter], JIT
    compiler, and static compiler

    (This is also the reason the JavaScript code follows the Mono convention of
    putting a space before the opening parenthesis of each function call.)

  * [colorForth][]

  * "[On Computable Numbers, with an Application to the
    Entscheidungsproblem][turing]"

    When I first started this project, I had been looking at this classic Alan
    Turing paper after having read the nice introduction from [*The annotated
    Turing*][annotated-turing].  It was fun to have this paper in mind while
    writing programs that didn't use named variables or scoping rules.

  * [M/o/Vfuscator](https://github.com/xoreaxeaxeax/movfuscator)

    > The M/o/Vfuscator ... compiles programs into "mov" instructions, and only
    > "mov" instructions.

## Similar projects

  * <https://github.com/mniip/BOOTSTRA>

    A project that includes both an 8086 assembler written using MS-DOS batch
    files and a Forth-like language implemented on top of that

  * <https://github.com/danistefanovic/build-your-own-x>

[colorForth]: https://web.archive.org/web/20160128005226/http://www.colorforth.com/inst.htm
[fc]: https://github.com/brendanzagaeski/0000/blob/master/fc.html
[fi]: https://github.com/brendanzagaeski/0000/blob/master/fi.html
[second-commit]: https://github.com/brendanzagaeski/0000/commit/0bdab6d0af4910b1cb96fefc8b1b3601f32c8d76
[mono-interpreter]: https://www.mono-project.com/news/2017/11/13/mono-interpreter/
[turing]: https://doi.org/10.1112%2Fplms%2Fs2-42.1.230
[annotated-turing]: https://lccn.loc.gov/2008022829

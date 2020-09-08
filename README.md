# Toy Forth-like compiler and interpreter

Toy compiler ([*fc.html*](fc.html)) and interpreter ([*fi.html*](fi.html)) for a
small Forth-like language

## Language examples

### Decrement five down to zero in a loop

Pretend that the compiled output for this code starts at virtual memory address
0x00000000.

```
0x00000005      0xFFFFFFFF      add             dup             0x00000010      jnz             exit            
```

### ASCII "Hello, world!"

```
0x6C6C6548      0x77202C6F      0x646C726F      0x00000A21      exit           
```

## Language

  * Fifteen primitive words: `dup`, `drop`, `over`, `push`, `pop`, `jz`, `jnz`,
    `jump`, `add`, `2*`, `2/`, `or`, `and`, `@`, `exit`
  * Hexadecimal number literals
  * Mandatory whitespace to pad each item to exactly 16 bytes
  * No control flow mechanism other than jumps to absolute memory addresses
  * No way to define new words
  * No relative or labeled jumps
  * No named variables
  * No way to write to memory other than the stacks
  * No way to include comments
  * No line breaks

## Compiler

  * Produces exactly 16 bytes of Intel 64 instructions per source item.
  * Takes its chances with outputting ELF headers and Intel 64 instructions
    itself rather than leveraging an existing assembler or compiler.
  * JavaScript bootstrap implementation uses funny low-level idioms like a stack
    and explicit loops so it can more closely imitate the self-hosting
    implementation.
  * Can compile itself.
  * Can compile the interpreter.
  * Implemented in a [single file](fc.html).

## Interpreter

  * [Differs from the compiler][second-commit] only by lines deleted from the
    parser and modifications to the primitive word definitions.
  * Can interpret itself.
  * Can interpret the compiler.

## Try it

### Compiler

 1. Download [*fc.html*][fc] and open it in a browser.
 2. When prompted, save the output to *fc0* on an x86_64 Linux system.
 3. ```sh
    chmod +x fc0 && ./fc0 > fc1
    chmod +x fc1 && ./fc1 > fc2
    chmod +x fc2 && ./fc2 > fc3
    chmod +x fc3 && ./fc3
    ```

#### Result

```
Hello, world!
```

#### Notes

*fc.html* uses the bootstrap compiler written in JavaScript to compile the
self-hosting compiler written in the Forth-like language to a Linux x86_64 ELF
executable *fc0*.  To test that *fc0* works correctly, *fc0* compiles its own
source code to produce *fc1*.  To test that *fc1* also works correctly, *fc1*
compiles the compiler written in the Forth-like language once more to produce
*fc2*.  *fc2* then compiles a final small "Hello, world!" program to produce
*fc3*.

### Interpreter

 1. Download [*fi.html*][fi] and open it in a browser.
 2. When prompted, save the output to a file and open it in a text viewer.

#### Result

```
Hello, world!
```

## Future possibilities

  * Simplify the parsing rules by switching from hexadecimal numbers to raw
    31-bit little-endian binary values.  Use the 32nd bit to distinguish between
    words and numbers.
  * Add an example of just-in-time compilation.
  * Experiment with making a more complete language that at least allows
    defining new words.

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

  * Curiosity about the overhead of interpreted versus compiled programs

## Similar projects

  * "[A Bestiary of Single-File Implementations of Programming
    Languages][b1fipl]"

  * <https://github.com/danistefanovic/build-your-own-x>

  * <https://github.com/mniip/BOOTSTRA>

    A project that includes both an 8086 assembler written using MS-DOS batch
    files and a Forth-like language implemented on top of that

[colorForth]: https://web.archive.org/web/20160128005226/http://www.colorforth.com/inst.htm
[fc]: ../../raw/main/fc.html
[fi]: ../../raw/main/fi.html
[second-commit]: ../../commit/0bdab6d0af4910b1cb96fefc8b1b3601f32c8d76
[mono-interpreter]: https://www.mono-project.com/news/2017/11/13/mono-interpreter/
[turing]: https://doi.org/10.1112%2Fplms%2Fs2-42.1.230
[annotated-turing]: https://lccn.loc.gov/2008022829
[b1fipl]: https://github.com/marcpaq/b1fipl

# ce-cpu

A toy 16-bit CPU ("TC16") with a tiny assembler, written in **[Ce++](https://github.com/Duffionis/Ce-)**
(a small, safe, all-Italian systems language that compiles to C).

It is the first real program built with Ce++ from *outside* the compiler's repository, so it also
shows what the language can do and what it still lacks.

    ./ce run src/principale.ce

The first run downloads the Ce++ compiler (pinned to a known-good version, see the `PIN` line in
`./ce`). You need `git`, `python3` and `gcc` or `clang`. The program runs its own self-tests and
prints `TC16: tutti i test ok` (exit code 0) if everything works.

## The CPU

8 registers (`r0`..`r7`, 16 bits), 512 words of memory, 16-bit instructions:

    tipo R:  oooo ddd sss 000000     opcode, destination register, source register
    tipo I:  oooo ddd iiiiiiiii      opcode, destination register, 9-bit constant

| op | name | effect |
|----|------|--------|
| 0 | NOP | nothing |
| 1 | LDI rd, imm | rd = imm |
| 2 | ADD rd, rs | rd = rd + rs (sets flags) |
| 3 | SUB rd, rs | rd = rd - rs (sets flags) |
| 4-6 | AND / OR / XOR rd, rs | bit operations |
| 7 | LD rd, rs | rd = memory[rs] |
| 8 | ST rd, rs | memory[rs] = rd |
| 9 | CMP rd, rs | flags from rd - rs, result discarded |
| 10 | JMP imm | pc = imm |
| 11 / 12 | JZ / JNZ imm | jump if zero flag set / clear |
| 13 | OUT rd | append rd to the output |
| 14 | HALT | stop |
| 15 | ADDI rd, imm | rd = rd + signed 9-bit imm |

Flags (zero, negative, carry) are a packed struct of three 1-bit fields.
Reading or writing outside memory, or running too long, gives an error value instead of crashing.

## Files

    src/cpu.ce         registers, flags, memory, one-step execution, run loop
    src/programmi.ce   assembler helpers (a.ldi(0, 5) ...), sample programs, disassembler
    src/principale.ce  self-test: Fibonacci, memory, arithmetic and bit ops, error cases
    ce                 wrapper that fetches and runs the pinned Ce++ compiler

## Ce++ features this program uses

- bit fields and a packed struct (`Flag`), `u16` arithmetic with defined wrap-around
- heap memory (`riserva` / `libera`) and slices with bounds checking
- generic library types: `Vettore<u16>` (output log), `Mappa<i32, testo>` (opcode names),
  `Risultato<T, E>` (errors without crashing), `Opzione<T>`
- reference parameters and method syntax: `c.passo()`, `a.emetti(w)`
- modules (`usa`)

## Writing your own program

Add a function to `src/programmi.ce` that takes `a: &Asm` and calls `a.ldi(...)`, `a.add(...)`,
`a.out(...)`, `a.halt()`; labels are just `a.n come u16`. Then load it in a test like
`prova_fibonacci` in `src/principale.ce`.

## Notes found while writing it (things Ce++ could do better)

- There is no `switch`: the instruction decoder is an `se` chain.
- `Flag` cannot be built from a raw word (no safe bit-cast between an integer and a packed struct yet).
- Operands print as `r0 / 0` in the disassembler for instructions that do not use them.

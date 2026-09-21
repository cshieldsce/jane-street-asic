<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

**Status: work in progress — the RTL is not written yet.**

Sutro is a small 16-bit soft processor with a serial I/O engine bolted onto it.
The instruction set is specified in `isa/isa.yaml`, which is the single source
of truth for the assembler, the simulator and the Verilog:

- 8 general-purpose 16-bit registers; `r0` reads as zero, `r7` is the link register
- 256-word program memory, 8-bit PC, free-running 32-bit cycle counter
- Z and C flags, written by the ALU ops and `ADDI`
- Fixed 16-bit instruction word in six formats (R, I, L, B, E, N)

The ISA covers ALU ops (`ADD`, `SUB`, `AND`, `OR`, `XOR`, `SLL`, `SRL`, `SRA`),
immediates (`ADDI`, `LI`, `LIH`), control flow (`B`, `JAL`, `JR`), timing
(`WAIT`, `WAITU`), and an engine interface (`PUSH`, `POP`, `CFG`, `RDE`) that
moves bytes in and out of the serial TX/RX FIFOs.

What is actually on the chip today is still the Tiny Tapeout template: `uo_out`
is the sum of `ui_in` and `uio_in`. The UART building blocks (baud generator,
shift register, bit counter, FSM) are being prototyped as Digital circuits under
`circuit/` before being turned into RTL.

## How to test

**Status: work in progress — the test below exercises the template design, not the core.**

The cocotb bench in `test/` drives the design through `test/tb.v`:

```
cd test
make clean
make
```

Today that checks the template adder: with `ui_in = 20` and `uio_in = 30`,
`uo_out` reads 50 after one clock.

Once the core lands, the plan is to hold the chip in reset, shift a program into
program memory over the serial link, release reset, and compare the bytes the
design pushes back out against the golden model driven from `isa/isa.yaml`.

## External hardware

None yet. The intended setup is a 3.3 V USB-to-serial adapter on the RX/TX pins.

# Single-Cycle MIPS32 processor

Verilog HDL implementation of a single-cycle MIPS32 processor, based on:

+ ["Computer Organization course"][1] by Muhamed F. Mudawar;
+ ["Digital Design and Computer Architecture"][2] by David Money Harris and Sarah L. Harris;
+ ["Computer Organization and Design"][3] by David Patterson and John Hennessy.

For further details and more detailed information about MIPS, refer to the sources, mentioned above.

## Architectual State and Instruction Set

Architecture of processor is determined by those basic things:

+ Register file - addressed set of high-speed storage cells, that are responsible for 32 32-bit general purpose registers for storing MIPS assembler commands;
+ Instruction memory - addressed block of memory, that contains sequence of commands, which are executed by processor one by one;
+ Data memory -  addressed block of memory, that contains information, that can be used by processor during execution of program;
+ Program Counter - addressed block of memory, that contains index of executable instruction.

On each cycle of the processor, the instruction with respective program counter index is being pulled out from the instruction memory. According to the instruction type and its parameters, the data is taken from register file and / or from data memory. After some actions, implemented on ALU (Arithmetic Logic Unit), new data is sent back to the register file and / or data memory and / or change the value of the program counter. So, the current state of the processor is determined by the values, written to the register file, data memory and program counter. Logic of transitioning between states is determined by the instruction memory values and the processor implementation.

## Register file

As it was mentioned before, register file is a block of memory, with 32x32 bits dimension. Operations with every register are implemented in the next sequence - data is loaded from instruction, stored in instruction memory (such as Rs, Rt, Rd - depends on the instruction format), and from data memory (RW). After operations with this data is completed, processor stores the result in memory and begins execution of the next instruction.

Every of 32 registers has its original index - from 0 to 31. Also registers are splitted into groups, they have own symbolic name and purpose. It is shown in this table:

| Name | Number | Purpose | 
|------|--------|---------|
| $0 | 0 | Always 0 (forced by hardware) | 
| $at | 1 | Temporary variable for assembler use | 
| $v0-$v1 | 2-3 | Result values of a procedure | 
| $a0-$a3 | 4-7 | Arguments of a  procedure | 
| $t0-$t7 | 8-15 | Temporary variables | 
| $s0-$s7 | 16-23 | Stored variables | 
| $t8-$t9 | 24-25 | More temporary variables | 
| $k0-$k1 | 26-27 | Reserved for OS kernel | 
| $gp | 28 | Global pointer (points to global data) | 
| $sp | 29 | Stack pointer (points to top of stack) | 
| $fp | 30 | Frame pointer (points to stack frame) | 
| $ra | 31 | Return address (used by jal for function call) | 

There are three MIPS instruction formats: 

+ **R-Type format. Both operands are registers.**

We can split R-Type format instructions into a few parts as described:

| OpCode | Rs | Rt | Rd | sa | Funct | Parts |
| ------ | -- | -- | -- | -- | ----- | ----- |
| 31..26 | 25..21 | 20..16 | 15..11 | 10..6 | 5..0 | Bits |

**- OpCode** stands for operation code (defines format of operation (R-Type, I-Type or J-Type, always 0 for R-Type instructions), 6 bits.

**- Rs** stands for the first register operand, 5 bits.

**- Rt** stands for the second register operand, 5 bits.

**- Rd** stands for destination register, 5 bits.

**- sa** stands for shift amount, 5 bits.

**- Funct** stands for function code (identifies the specific R-Type instructions), 6 bits.

Example of R-Type instruction:

```
add $5, $4, $3
```

Processor will calculate and write sum of values, stored in registers $4 and $3 into register $5.

+ **I-Type. One of the operands is register, other is an immediate value.**

We can split I-Type format instructions into a few parts as described:

| OpCode | Rs | Rt | immediate | Parts |
| ------ | -- | -- | --------- | ----- |
| 31..26 | 25..21 | 20..16 | 15..0 | Bits |

**- OpCode** stands for operation code (defines format of operation (R-Type, I-Type or J-Type), 6 bits.

**- Rs** stands for the register, containing base address, 5 bits.

**- Rt** stands for destination/source register, 5 bits.

**- immediate** stands for value or offset, 16 bit.

Example of I-Type instruction:

```
xori $15, $10, 27
```

Processor will calculate and write result of bitwise XOR between value, stored in registers $10 and immediate value 27.

+ **J-Type. Have a label, sort of address to which program counter is transferred.**

We can split R-Type format instructions into a few parts as described:

| OpCode | Target address | Parts |
| ------ | -------------- | ----- |
| 31..26 | 25..0 | Bits |

**- OpCode** stands for operation code (defines format of operation (R-Type, I-Type or J-Type), 6 bits.

**- Target address** stands for the register, containing base address, 5 bits.

Example of J-Type instruction:

```
beq $17, $16, label
or $18, $17, $16
label: and $18, $17, $16
```

Processor will branch to *label* if $17 == $16, else it will execute next instruction.

In this realisation of single-cycle MIPS32 processor there are supported 22 assembler instructions of different formats.

R-Type Format instructions:

+ ADD
+ SUB
+ AND
+ OR
+ XOR
+ NOR
+ SLT
+ SLL
+ SRL
+ SRA
+ ROR (created own pseudo-instruction)
+ ROL (created own pseudo-instruction)

I-Type Format instructions:

+ ADDI
+ SLTI
+ ANDI
+ ORI
+ XORI
+ LW
+ SW

J-Type Format instructions:

+ BNE
+ BEQ
+ J

## Instruction memory

In this realisation of MIPS, instruction memory initialize binary file ```ROM_FILE.bin```, that contains sequence of commands, desribed in assembler file. These commands presented in hexadecimal values, so that machine would be able to understand them. Conversion from assembler code into hexadecimal format is completed using [MIPS Converter](https://www.eg.bucknell.edu/~csci320/mips_web/). Also, this service can convert hexadeciaml value into assembler code. Note, that you have to convert your register numeric value into its name. For example:

Original assembler instruction:

```
or $18, $17, $16
```

Converted assembler instruction:

```
or $s2 $s1 $s0
```

Its hehadecimal value:

```
0x02309025
```

## Program Counter

Let's see, how following program will look like in instruction memory. For example, program is:

```
#Assembler code      Comment                    PC   Hex value
addi $2,  $0, 5      # initialize $2 = 5  	@0   20020005
addi $3,  $0, 12     # initialize $3 = 12 	@4   2003000C
addi $7,  $3, -9     # initialize $7 = 3  	@8   2067FFF7
or   $4,  $7, $2     # $4 <= 3 or 5 = 7   	@C   00E22025
```

How it will loke lie in insrtuction memory:

```
0x00000000: 00100000 00000010 00000000 00000101
0x00000004: 00100000 00000011 00000000 00001100
0x00000008: 00100000 01100111 11111111 11110111
0x0000000C: 00000000 11100010 00100000 00100101
```

When processor starts working, it will take first 4 bites from instruction memory, and executes it as one command. After execution it will take the next 4 bites from instruction memory (in case J-Type format instructions, will take 4 bites from the point, where it will bracnh). Program counter is needed for processor because it contains address os instruction, that is executed during current cycle. 

[1]: https://faculty.kfupm.edu.sa/COE/mudawar/coe301/lectures/index.htm
[2]: https://moodlearn.ariel.ac.il/pluginfile.php/1649872/mod_resource/content/0/%5BDavid_Harris%2C_Sarah_Harris%5D_Digital_Design_and_Co%28b-ok.xyz%29.pdf
[3]: https://ict.iitk.ac.in/wp-content/uploads/CS422-Computer-Architecture-ComputerOrganizationAndDesign5thEdition2014.pdf

# ALU Design for 32-bit Instruction Set Architectures

## Project Environment & Level

- **Design Language**: Verilog Hardware Description Language (HDL)
- **Simulation Environment**: Vivado 2024.1
- **Test Environment: LS-CPU-EXB-002 Experimental Box, equipped with Xilinx Artix-7 xc7a200tfbg676-2**

## Project Description

   A 32-bit Arithmetic Logic Unit (ALU) that supports MIPS 32-bit ISA.

### Basic Functions:

1. **Verilog HDL**: Use Verilog to describe the hardware.
2. **Basic Operations**: The ALU supports the following operations:
   - Addition(Carry-Lookahead Adder)
   - Shift operations
   - Logical operations (AND, OR, etc.)
   - Multiplication(Serial Multiplier)
   - Division(Restoring Remainder Divider)
3. **Simulation & Implementation**:
   - The design is simulatable in Vivado and able to generate a bitstream for deployment on an FPGA board.
4. **Test Cases**:
   - Provide several test cases. These are embedded into the design and will be explained in further detail later.

### Control Signal Encoding:

| Operation              | Control Signal (alu_control)  | Description                                                                            |
|------------------------|-------------------------------|----------------------------------------------------------------------------------------|
| Division               | 14'b10000000000000            | Perform division (alu_src1 / alu_src2)                                                 |
| Multiplication         | 14'b01000000000000            | Perform multiplication (alu_src1 * alu_src2)                                           |
| Addition               | 14'b00100000000000            | Perform addition (alu_src1 + alu_src2)                                                 |
| Subtraction            | 14'b00010000000000            | Perform subtraction (alu_src1 - alu_src2)                                              |
| Signed Less Than       | 14'b00001000000000            | Check if alu_src1 is less than alu_src2, signed comparison                             |
| Unsigned Less Than     | 14'b00000100000000            | Check if alu_src1 is less than alu_src2, unsigned comparison                           |
| Bitwise AND            | 14'b00000010000000            | Perform bitwise AND operation on alu_src1 and alu_src2                                 |
| Bitwise NOR            | 14'b00000001000000            | Perform bitwise NOR operation on alu_src1 and alu_src2                                 |
| Bitwise OR             | 14'b00000000100000            | Perform bitwise OR operation on alu_src1 and alu_src2                                  |
| Bitwise XOR            | 14'b00000000010000            | Perform bitwise XOR operation on alu_src1 and alu_src2                                 |
| Logical Left Shift     | 14'b00000000001000            | Perform logical left shift on alu_src1 (shift amount determined by alu_src2)           |
| Logical Right Shift    | 14'b00000000000100            | Perform logical right shift on alu_src1 (shift amount determined by alu_src2)          |
| Arithmetic Right Shift | 14'b00000000000010            | Perform arithmetic right shift on alu_src1 (shift amount determined by alu_src2)       |
| Load Upper Immediate   | 14'b00000000000001            | Load upper 16 bits of alu_src2 into alu_src1, other parts zeroed, result in lui_result |

### Test Case Description：

1. **Test Cases Description**:
   - **0 ns - 5 ns**: Initialization
     - Signal values: 
       - `alu_control = 14'b0` (all zeros, 14-bit control signal, no operation activated)
       - `alu_src1 = 32'h0` (0)
       - `alu_src2 = 32'h0` (0)
       - `alu_result`: initially `32'h0` (usually 0)
     - Operation: No specific operation. This is the initialization stage of the simulation, where all signals are initialized to 0 to avoid undefined states (XXXX).

   - **5 ns - 10 ns**: First Multiplication
     - Signal values:
       - `alu_control = 14'b01_0000_0000_0000`
       - `alu_src1 = 32'hc` (12)
       - `alu_src2 = 32'hf` (15)
       - `alu_result`: after 5 ns becomes `32'hb4` (12 * 15 = 180)
     - Operation: The ALU performs the multiplication operation, calculating `0xc * 0xf`. The result is output after propagation delay.

   - **10 ns - 15 ns**: Second Multiplication
     - Signal values:
       - `alu_control = 14'b01_0000_0000_0000`
       - `alu_src1 = 32'h457` (1111)
       - `alu_src2 = 32'h8ae` (2222)
       - `alu_result`: after 10 ns becomes `32'h25ab22` (1111 * 2222 = 2468642)
     - Operation: The ALU performs the multiplication operation, calculating `0x457 * 0x8ae`. The result is output after propagation delay.

   - **15 ns - 20 ns**: Addition
     - Signal values:
       - `alu_control = 14'b00_1000_0000_0000`
       - `alu_src1 = 32'h1` (1)
       - `alu_src2 = 32'hffffffff` (-1 or 4294967295)
       - `alu_result`: after 15 ns becomes `32'h0` (1 + (-1) = 0)
     - Operation: The ALU performs the addition operation, calculating `0x1 + 0xffffffff`. The result is 0 (signed overflow).

   - **20 ns - 25 ns**: Subtraction
     - Signal values:
       - `alu_control = 14'b00_0100_0000_0000`
       - `alu_src1 = 32'h1` (1)
       - `alu_src2 = 32'h2` (2)
       - `alu_result`: after 20 ns becomes `32'hffffffff` (1 - 2 = -1)
     - Operation: The ALU performs the subtraction operation, calculating `0x1 - 0x2`. The result is -1 (32-bit two's complement).

   - **25 ns - 30 ns**: Signed Comparison (Set if less)
     - Signal values:
       - `alu_control = 14'b00_0010_0000_0000`
       - `alu_src1 = 32'h1` (1)
       - `alu_src2 = 32'h2` (2)
       - `alu_result`: after 25 ns becomes `32'h1` (1 < 2 is true)
     - Operation: The ALU performs a signed comparison, checking if `0x1 < 0x2`. The result is 1.

   - **30 ns - 35 ns**: Unsigned Comparison
     - Signal values:
       - `alu_control = 14'b00_0001_0000_0000`
       - `alu_src1 = 32'h1` (1)
       - `alu_src2 = 32'h2` (2)
       - `alu_result`: after 30 ns becomes `32'h1` (1 < 2 is true)
     - Operation: The ALU performs an unsigned comparison, checking if `0x1 < 0x2`. The result is 1.

   - **35 ns - 40 ns**: Bitwise AND
     - Signal values:
       - `alu_control = 14'b00_0000_1000_0000`
       - `alu_src1 = 32'h12345678` (0x12345678)
       - `alu_src2 = 32'hf0f0f0f0` (0xf0f0f0f0)
       - `alu_result`: after 35 ns becomes `32'h10305070` (0x12345678 & 0xf0f0f0f0)
     - Operation: The ALU performs a bitwise AND operation, calculating `0x12345678 & 0xf0f0f0f0`.

   - **40 ns - 45 ns**: Bitwise NOR
     - Signal values:
       - `alu_control = 14'b00_0000_0100_0000`
       - `alu_src1 = 32'he` (0xe)
       - `alu_src2 = 32'h1` (1)
       - `alu_result`: after 40 ns becomes `32'hfffffff0` (~(0xe | 1))
     - Operation: The ALU performs a bitwise NOR operation, calculating `~(0xe | 1)`.

   - **45 ns - 50 ns**: Bitwise OR
     - Signal values:
       - `alu_control = 14'b00_0000_0010_0000`
       - `alu_src1 = 32'he` (0xe)
       - `alu_src2 = 32'h1` (1)
       - `alu_result`: after 45 ns becomes `32'hf` (0xe | 1)
     - Operation: The ALU performs a bitwise OR operation, calculating `0xe | 1`.

   - **50 ns - 55 ns**: Bitwise XOR
     - Signal values:
       - `alu_control = 14'b00_0000_0001_0000`
       - `alu_src1 = 32'ha` (0xa)
       - `alu_src2 = 32'h5` (0x5)
       - `alu_result`: after 50 ns becomes `32'hf` (0xa ^ 0x5)
     - Operation: The ALU performs a bitwise XOR operation, calculating `0xa ^ 0x5`.

   - **55 ns - 60 ns**: Logical Left Shift
     - Signal values:
       - `alu_control = 14'b00_0000_0000_1000`
       - `alu_src1 = 32'h4` (4)
       - `alu_src2 = 32'hf` (0xf)
       - `alu_result`: after 55 ns becomes `32'hf0` (0xf << 4)
     - Operation: The ALU performs a logical left shift, calculating `0xf << 4`.

   - **60 ns - 65 ns**: Logical Right Shift
     - Signal values:
       - `alu_control = 14'b00_0000_0000_0100`
       - `alu_src1 = 32'h4` (4)
       - `alu_src2 = 32'hf0` (0xf0)
       - `alu_result`: after 60 ns becomes `32'hf` (0xf0 >> 4)
     - Operation: The ALU performs a logical right shift, calculating `0xf0 >> 4`.

   - **65 ns - 70 ns**: Arithmetic Right Shift
     - Signal values:
       - `alu_control = 14'b00_0000_0000_0010`
       - `alu_src1 = 32'h4` (4)
       - `alu_src2 = 32'hf0000000` (0xf0000000)
       - `alu_result`: after 65 ns becomes `32'hff000000` (0xf0000000 >>> 4)
     - Operation: The ALU performs an arithmetic right shift, calculating `0xf0000000 >>> 4`, preserving the sign bit.

   - **70 ns - 75 ns**: Load Upper Immediate
     - Signal values:
       - `alu_control = 14'b00_0000_0000_0001`
       - `alu_src1: unused` (stay 0x4 but does not affect result)
       - `alu_src2 = 32'hbfc0` (0xbfc0)
       - `alu_result`: after 70 ns becomes `32'hbfc00000` (0xbfc0 << 16)
     - Operation: The ALU performs the load upper immediate, shifting `0xbfc0` left by 16 bits, filling lower bits with 0.

   - **75 ns - 80 ns**: First Division
     - Signal values:
       - `alu_control = 14'b10_0000_0000_0000`
       - `alu_src1 = 32'ha` (10)
       - `alu_src2 = 32'h3` (3)
       - `alu_result`: after 75 ns becomes `32'h3` (10 / 3 = 3)
     - Operation: The ALU performs a division operation, calculating `0xa / 0x3`, the result is the integer part `3`.

   - **80 ns - 85 ns**: Second Division
     - Signal values:
       - `alu_control = 14'b10_0000_0000_0000`
       - `alu_src1 = 32'h14` (20)
       - `alu_src2 = 32'h4` (4)
       - `alu_result`: after 80 ns becomes `32'h5` (20 / 4 = 5)
     - Operation: The ALU performs a division operation, calculating `0x14 / 0x4`, the result is `5`.

   - **85 ns - 95 ns**: End Wait
     - Signal values:
       - `alu_control = 14'b10_0000_0000_0000` (remain unchanged)
       - `alu_src1 = 32'h14` (remain unchanged)
       - `alu_src2 = 32'h4` (remain unchanged)
       - `alu_result`: remains `32'h5`
     - Operation: No new operations, waiting for 10 ns to complete the simulation.

   - **95 ns**: End of Simulation
     - Signal values: No changes.
     - Operation: The simulation ends via `$finish`.

## How to Build

1. Clone this repository:
   ```bash
   git clone https://github.com/your-username/alu-project.git

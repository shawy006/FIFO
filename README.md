
# Synchronous and Asynchronous FIFO Design in Verilog HDL


## Project Overview

This project implements both synchronous and asynchronous FIFO (First-In-First-Out) modules in Verilog HDL. The synchronous FIFO operates within a single clock domain, while the asynchronous FIFO supports different read and write clock domains, making it suitable for applications requiring cross-domain data transfer.

Each FIFO module is 8 bits wide and has a depth of 64. The design includes `read` and `write` enable control signals, along with `empty` and `full` flags to indicate the FIFO status.

## Features

- **8-bit Data Width, 64-depth FIFO**: Each FIFO module supports 8-bit data and can hold up to 64 entries.
- **Control Signals**: `wr_en` (write enable) and `rd_en` (read enable) signals for controlling data writes and reads.
- **Status Flags**: `empty` and `full` flags to indicate when the FIFO is empty or full.
- **Synchronous FIFO**: Operates in a single clock domain for both read and write operations.
- **Asynchronous FIFO**: Supports different clock domains for read and write operations, using Gray code counters for pointer synchronization.

## Design Details

### 1. Synchronous FIFO
- **Clock**: Operates on a single clock signal (`clk`).
- **Read and Write Operations**: Controlled by `wr_en` and `rd_en` signals. Data is written when `wr_en` is high and read when `rd_en` is high, provided that the FIFO is not full or empty.
- **Pointer Management**: A single `wr_ptr` (write pointer) and `rd_ptr` (read pointer) manage the FIFO's memory array.
- **Empty and Full Flags**: `empty` flag is high when there are no entries, and `full` flag is high when the FIFO reaches maximum depth.

### 2. Asynchronous FIFO
- **Separate Clocks**: `wr_clk` for write operations and `rd_clk` for read operations, allowing for cross-clock domain data transfers.
- **Gray Code Synchronization**: Uses Gray code counters to manage read and write pointers, enabling safe data transfer between the different clock domains.
- **Empty and Full Flags**: `empty` flag is high when the read pointer catches up to the synchronized write pointer, and `full` flag is high when the write pointer catches up to the synchronized read pointer.

## File Structure

- **`sync_fifo.v`**: Synchronous FIFO Verilog module
- **`async_fifo.v`**: Asynchronous FIFO Verilog module
- **`tb_sync_fifo.v`**: Testbench for synchronous FIFO
- **`tb_async_fifo.v`**: Testbench for asynchronous FIFO

## Usage

1. **Compilation**: Use a Verilog simulator (e.g., ModelSim, Xilinx Vivado) to compile the Verilog modules and run the testbenches.
2. **Simulation**: 
   - Run `tb_sync_fifo.v` to test the synchronous FIFO.
   - Run `tb_async_fifo.v` to test the asynchronous FIFO.
3. **Customization**: Modify the `WIDTH` and `DEPTH` parameters in the Verilog code to change the data width and depth of the FIFO.

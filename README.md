
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


## Code
Below is the Verilog code for each module and testbench as specified.

---

### `sync_fifo.v` - Synchronous FIFO Verilog Module

```verilog
module sync_fifo #(
    parameter WIDTH = 8,   // Data width
    parameter DEPTH = 64   // FIFO depth
)(
    input clk,             // Clock signal
    input rst,             // Reset signal
    input wr_en,           // Write enable
    input rd_en,           // Read enable
    input [WIDTH-1:0] data_in,  // Data input
    output reg [WIDTH-1:0] data_out,  // Data output
    output reg empty,      // FIFO empty flag
    output reg full        // FIFO full flag
);

    reg [WIDTH-1:0] mem [0:DEPTH-1];  // FIFO memory array
    reg [5:0] wr_ptr = 0;             // Write pointer
    reg [5:0] rd_ptr = 0;             // Read pointer
    reg [6:0] count = 0;              // Element counter

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            wr_ptr <= 0;
            rd_ptr <= 0;
            count <= 0;
            empty <= 1;
            full <= 0;
        end else begin
            // Write operation
            if (wr_en && !full) begin
                mem[wr_ptr] <= data_in;
                wr_ptr <= (wr_ptr + 1) % DEPTH;
                count <= count + 1;
            end

            // Read operation
            if (rd_en && !empty) begin
                data_out <= mem[rd_ptr];
                rd_ptr <= (rd_ptr + 1) % DEPTH;
                count <= count - 1;
            end

            // Update flags
            empty <= (count == 0);
            full <= (count == DEPTH);
        end
    end

endmodule
```

---

### `async_fifo.v` - Asynchronous FIFO Verilog Module

```verilog
module async_fifo #(
    parameter WIDTH = 8,   // Data width
    parameter DEPTH = 64   // FIFO depth
)(
    input wr_clk,          // Write clock
    input rd_clk,          // Read clock
    input rst,             // Reset signal
    input wr_en,           // Write enable
    input rd_en,           // Read enable
    input [WIDTH-1:0] data_in,  // Data input
    output reg [WIDTH-1:0] data_out,  // Data output
    output reg empty,      // FIFO empty flag
    output reg full        // FIFO full flag
);

    reg [WIDTH-1:0] mem [0:DEPTH-1];  // FIFO memory array
    reg [5:0] wr_ptr = 0;             // Write pointer (binary)
    reg [5:0] rd_ptr = 0;             // Read pointer (binary)
    reg [5:0] wr_ptr_gray = 0;        // Write pointer (Gray code)
    reg [5:0] rd_ptr_gray = 0;        // Read pointer (Gray code)

    reg [5:0] wr_ptr_gray_sync, rd_ptr_gray_sync;  // Synchronized pointers

    always @(posedge wr_clk or posedge rst) begin
        if (rst) begin
            wr_ptr <= 0;
            wr_ptr_gray <= 0;
            full <= 0;
        end else if (wr_en && !full) begin
            mem[wr_ptr] <= data_in;
            wr_ptr <= (wr_ptr + 1) % DEPTH;
            wr_ptr_gray <= (wr_ptr >> 1) ^ wr_ptr;  // Binary to Gray code
        end
    end

    always @(posedge rd_clk or posedge rst) begin
        if (rst) begin
            rd_ptr <= 0;
            rd_ptr_gray <= 0;
            empty <= 1;
        end else if (rd_en && !empty) begin
            data_out <= mem[rd_ptr];
            rd_ptr <= (rd_ptr + 1) % DEPTH;
            rd_ptr_gray <= (rd_ptr >> 1) ^ rd_ptr;  // Binary to Gray code
        end
    end

    // Synchronize pointers
    always @(posedge rd_clk or posedge rst)
        if (rst) wr_ptr_gray_sync <= 0;
        else wr_ptr_gray_sync <= wr_ptr_gray;

    always @(posedge wr_clk or posedge rst)
        if (rst) rd_ptr_gray_sync <= 0;
        else rd_ptr_gray_sync <= rd_ptr_gray;

    // Empty and Full flags
    always @* begin
        empty = (wr_ptr_gray_sync == rd_ptr_gray);
        full = (wr_ptr_gray == {~rd_ptr_gray[5:4], rd_ptr_gray[3:0]});
    end

endmodule
```

---

### `tb_sync_fifo.v` - Testbench for Synchronous FIFO

```verilog
module tb_sync_fifo();

    reg clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire empty;
    wire full;

    // Instantiate the FIFO module
    sync_fifo fifo (
        .clk(clk),
        .rst(rst),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .data_in(data_in),
        .data_out(data_out),
        .empty(empty),
        .full(full)
    );

    // Clock generation
    always #5 clk = ~clk;

    initial begin
        // Initialize signals
        clk = 0;
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h00;

        // Reset FIFO
        #10 rst = 0;

        // Write data into FIFO
        repeat (5) begin
            #10 wr_en = 1;
            data_in = data_in + 8'h01;
        end
        wr_en = 0;

        // Read data from FIFO
        #20 rd_en = 1;
        repeat (5) #10;
        rd_en = 0;

        // End simulation
        #10 $finish;
    end

endmodule
```

---

### `tb_async_fifo.v` - Testbench for Asynchronous FIFO

```verilog
module tb_async_fifo();

    reg wr_clk;
    reg rd_clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire empty;
    wire full;

    // Instantiate the FIFO module
    async_fifo fifo (
        .wr_clk(wr_clk),
        .rd_clk(rd_clk),
        .rst(rst),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .data_in(data_in),
        .data_out(data_out),
        .empty(empty),
        .full(full)
    );

    // Clock generation for write and read clocks
    always #5 wr_clk = ~wr_clk;
    always #7 rd_clk = ~rd_clk;

    initial begin
        // Initialize signals
        wr_clk = 0;
        rd_clk = 0;
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h00;

        // Reset FIFO
        #10 rst = 0;

        // Write data into FIFO
        repeat (5) begin
            #10 wr_en = 1;
            data_in = data_in + 8'h01;
        end
        wr_en = 0;

        // Read data from FIFO
        #20 rd_en = 1;
        repeat (5) #10;
        rd_en = 0;

        // End simulation
        #10 $finish;
    end

endmodule
```

---

This code covers the core modules and testbenches for both synchronous and asynchronous FIFO designs. Each testbench initializes signals, applies resets, and performs read and write operations to validate functionality. Adjust the simulation parameters as needed for further testing and verification.





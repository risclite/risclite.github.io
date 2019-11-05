## How to parameterize SSRV to balance performance and area

-------------------------------------

Four multiple-in, multiple-out buffers are embedded in four different modules. Each buffer has its own role to regulate instructions. A multiple-in, multiple-out buffer has 3 parameters: IN_LEN, OUT_LEN and BUF_LEN. This chapter tells you how to define parameters of each buffer. Please open rtl/define_para.v and find definitions of these parameters.

* The instrbits buffer

This buffer is to accumulate bits of instruction steam and make sure there are enough bits to make up “FETCH_LEN” instructions.
The IN_LEN of the buffer is the width of instruction memory bus. It is parameterized as “BUS_LEN”.  The width of the instruction memory bus of RV32IMC is XLEN bits, here defined as 32 bits. So, the bit length of IN_LEN is BUS_LEN\*XLEN.

The BUF_LEN attribute of the buffer is based on bits. Its direct Verilog definition is “INBUF_LEN”. It means how many “imem_rdata” will be stored. The length of “imem_rdata” is “BUS_LEN”, which indicates how many words (32 bits) are fetched in the same cycle.  The capacity of the buffer is INBUF_LEN\*BUS_LEN\*XLEN bits.

The OUT_LEN of the buffer is to count how many instructions to output. It is not a fixed number of bits. Its definition is “FETCH_LEN”, which determines how many instructions is parallel to be dealt with.

|Attributes  |Definition|	What it means|
|------------|----------|----------------|
|Multiple-in |BUS_LEN   |BUS_LEN\*XLEN bits | 
|Capacity    |INBUF_LEN	|INBUF_LEN\*BUS_LEN\*XLEN bits |
Multiple-out |FETCH_LEN	|FETCH_LEN instructions|


* The schedule buffer

The main buffer of SSRV is used to accumulate instructions and try to send multiple instructions to be executed.

The IN_LEN attribute of the buffer is “FETCH_LEN”, which means how many instructions could be supplied by its servant buffer.

The BUF_LEN attribute of the buffer is “SDBUF_LEN”. It is the maximum number to be evaluated at the same cycle.

The OUT_LEN attribute of the buffer is “EXEC_LEN”. It defines how many ALU modules to be instantiated.

|Attributes  |Definition|	What it means|
|------------|----------|----------------|
|Multiple-in |FETCH_LEN |instruction number | 
|Capacity    |SDBUF_LEN	|instruction number |
Multiple-out |EXEC_LEN	|instruction number |

* The mprf buffer

This buffer keeps the operation result of ALU instructions and make sure only instructions ahead of the current active MEM instruction are allowed into the register file.

The IN_LEN attribute is less than EXEC_LEN. It depends on how many ALU instructions there are in the execution list.

The BUF_LEN attribute is defined as “RFBUF_LEN”. To increase it will make this buffer contains more ALU instructions.

The OUT_LEN attribute is “WRRG_LEN”. It can be assigned to “EXEC_LEN”.

|Attributes  |Definition|	What it means|
|------------|----------|----------------|
|Multiple-in |EXEC_LEN  |instruction number | 
|Capacity    |RFBUF_LEN	|instruction number |
Multiple-out |WRRG_LEN	|instruction number |

* The membuf buffer

This buffer is composed of queued MEM instructions and in every cycle the nearest MEM instruction becomes the current active one, who has the authority to operate the data bus, get the mul/div result, or exchange CSR and general-purpose registers.

The IN_LEN attribute is less than EXEC_LEN. It depends on how many MEM instructions there are in the execution list.

The BUF_LEN attribute is defined as “RFBUF_LEN”. To increase it will make this buffer contains more MEM instructions.

The OUT_LEN attribute is 1. There is only 1 MEM instruction to be retired in each cycle.

|Attributes  |Definition|	What it means|
|------------|----------|----------------|
|Multiple-in |EXEC_LEN  |instruction number | 
|Capacity    |MMBUF_LEN	|instruction number |
Multiple-out |1      	|instruction number |

Below are the 4 important parameters:

* BUS_LEN

* FETCH_LEN

* SDBUF_LEN

* EXEC_LEN

They will determine how many instructions are parallel on the same cycle. The default configuration is suggested as BUS_LEN==FETCH_LEN==EXEC_LEN. “BUS_LEN” as a data bus width, is necessary to be exponential of 2, such 1, 2, 4, 8 etc.

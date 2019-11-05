## Instructions

-------------------------------------

SSRV has two buffers to handle two different kinds of instructions. One is for alu instructions in the "mprf" module, which are OP or OP-IMM instructions of RV32IC. The other is for mem instructions in the "membuf" module, which are ones related with memory operation.
It is not possible to retire two mem instructions when there is only one data bus. The buffer of "membuf" module gathers some mem instructions and issue one of them in every cycle. Each of them is possible to cause an exception when data bus reports error. The mem instructions could be called "major" instructions.

The alu instructions could be called "minor" instructions. When they have a chance to be executed, their destination register numbers and updating data are queued in the buffer, until their previous mem instructions are all retired. It is necessary to make sure no invasion to the register file from its following alu instructions when an exception occurs.

|     |alu0	|alu1  |mem0    |alu2	|alu3	|mem1	|alu4  |
|-----|-----|------|--------|-------|-------|-------|------|
|order|	0   |	0  |	1   |	1   |1      |	2   | 2    |

To distinguish alu instructions, there is a method that is to give every instructions one order. When the current instruction is one of mem instructions, its order will plus one, or keep the same. The first mem instructions of the "membuf" buffer will always have an order: 1, which is the current instruction operating the data bus. Instructions, which have the order 0, will be allowed to update the register file. If the current mem instruction is reporting successful, the order of each alu instruction will decrease one until it reaches 0. That a mem instruction retires successfully will bring more alu instructions with "0" order.

Every cycle, SSRV tries to retire one mem instruction and multiple alu instructions. According to their empty space of both buffers, "schedule" module will issue indefinite instructions and try to fill these two buffers.

So, there are two basic kinds of instructions:

* alu: only related with the registers:R1~R31, includes: OP/OP-IMM of RV32IC

* mem: load and store instructions

However, there are two kinds of instructions, which will have to be dealed with as the same as mem instructions.They are:

* mul: multiply-divide instructions.

* csr: exchange data between CSR and general-purpose registers.

The mul instructions wll have multiple cycle to be executed. Each mem instruction will also need varied cycles to complete. In case of that, the mul instruction can be treated as one special memory-loading instruction. 

The csr instruction will exchange one general-purpose register with one CSR register. It could be treated as one special mem instruction: loading and storing in the same cycle.

These two kinds of instructions will have their own exclusive cycle to complete operation. Below, MEM means three kind of instructions: mem, mul and csr, which are dedicated to the “membuf” module. ALU means alu instructions, whose operation result enters the “mprf” module.
 
MEM and ALU instructions can be scheduled out-of-order, which means its following instructions could be issued before it does. Instructions introduced next are not allowed because its following ones are disabled permanently or temporarily.

* err : error occurs when fetching this instruction, treated as a special instruction.

* illegal: fetching successfully, but it does not belong any defined RV32 instructions.

* fencei:  one kind of RV32I instructions

* fence : one kind of RV32I instructions.

* sys : system instructions, often related to CSR, such as: break, call.

* jalr : jump instructions, target address is provided by one general-purpose register.

* jal : jump instructions, target address is related to PC.

* jcond: conditional jump instructions.

These 8 kinds of instructions are named “SPECIAL” instructions.

When the “schedule” module initializes a scheduling process between “SDBUF_LEN” number of instructions, SPECIAL instructions should be treated as the last one because its following ones cannot be executed unless they are retired. Also, it is not possible that two of SPECIAL instructions exist in the same list of the “schedule” module.

SPECIAL instructions can have two processing styles just like MEM and ALU ones do. 

“jalr, jal, jcond” are ALU-like SPECIAL instructions. One of them could change PC to its target address and make instructions of the target address queued after instructions ahead of this jump instruction. It can trigger jump operation before some MEM instructions ahead of it are queued.

The others are MEM-like SPECIAL instructions. One of them will wait until all MEM instructions ahead of it are retired successfully. It involves with changing some CSRs and it is not revoked. So, it should be treated as one of MEM instruction. 


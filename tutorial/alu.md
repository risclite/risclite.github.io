
## alu/alu_with_jump

-------------------------------------


This module will deal with an instruction and produce necessary signals for the next corresponding modules.

When an instruction arrives, it includes validity, instruction word, parameters and PC. First of all, it will fetch its Rs0 and Rs1 operands.

If it is an ALU instruction, a “rd_sel” signal will indicate which register is activated to modify and “rd_data” is the new data.

If it is a MEM instruction, it will give the below signals:

* mem_vld: a validity signal.

* mem_para: load and store method signals, it indicates how to operate the data bus. If it is a loading operation, it includes the destination register and byte,half-word, word selection signals.

* mem_addr: the address signal of the loading and storing operation.

* mem_wdata: the writing data of the storing operation.

These are necessary signals for operation of the data bus. When they reaches the “membuf” module, they will be queued  and dealt with one by one.

The last ALU module is different with the others. It will have an extra function: to deal with jump instructions. It will produce two signals: branch_vld and branch_pc.

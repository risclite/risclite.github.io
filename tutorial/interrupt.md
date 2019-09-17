## Interrupts and exceptions

-------------------------------------

As you know, SSRV supports little system functions for now. SSRV is engaged to be a fundamental framework of CPU cores. It can provide a solution of super-scalar and out-of-order, which is essential of high performance. 

SSRV is easy to support interrupts and exceptions. Here it will be detailed how SSRV could deal with them.

* instruction exceptions

If an instruction-fetching request has an error response, this condition will be treated as a special instruction: err. An err instruction belongs to SPECIAL instruction, which will be transferred to the “sys_csr” module. This module will invoke a jump operation, whose target address is one of CSR registers.

The err instruction as a SPECIAL one, is the last one of the “schedule” buffer. If it brings a jump operation, the signal “jump_vld” will clear input instructions from “instrbits”.

* data memory exceptions

The nearest MEM instruction is the current active one, which has the whole control of data memory. If data memory reports an exception, it should be from the nearest MEM instruction. 

If this MEM instruction causes an exception, any following instruction should be cleared from the pipeline. It can be implemented by assert the signal “clear_pipeline”. Any ALU instructions, whose order parameter is not zero, should be the following ones. Asserting “clear_pipeline” will eliminate the operation result of these instructions. They will never have a chance to enter the register file.

The “clear_pipeline” signal will also apply to the “schedule” module. Any instructions whose order is not zero will be eliminated from the buffer. 

When the error signal of data memory accesses “sys_csr” module, this module will give a jump operation to the target address of this exception.

* interrupt
An interrupt could happen on any cycle. If it happens, SSRV has to provide an interrupt address to halt the pipeline. When the interrupt service program ends, it should resume execution from the interrupt address.

At every cycle, SSRV will pick an instruction as the interrupt instruction. If there is at least one in the “membuf” buffer, the nearest MEM instruction is an ideal one. It will abort its data memory operation and fire a jump operation to the interrupt service entry address.

If the “membuf” buffer is empty, SSRV will search a MEM or SPECIAL instruction in the “schedule” buffer. If it exists, this instruction will be the interrupt instruction. If it could not find one and a jump instruction are writing to PC, the target address of the jump instruction is the interrupt instruction. If there is no jump operation, the base address of the “instrbits” buffer is the interrupt instruction.

An interrupt will also need to assert “clear_pipeline”. Instruction with the order 0 will be kept in the pipeline because in the next few cycles, they will be retired without any problem. “clear_pipeline” will clear any instructions with the order, which is not zero.

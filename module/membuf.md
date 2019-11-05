## membuf

-------------------------------------

Every MEM instruction from “alu” module is queued in the buffer of this module. There is only one MEM instruction as a current active one. It could operate the data bus, or write mul/div result to the register file, or start exchange of CSR and general-purpose registers.

Firstly, signals from “alu” modules are screened to exclude empty ones. After that, they are queued behind ones stored.

To improve the efficiency of mul/div operations, it is necessary to find out one mul instruction when it is queued between the buffer and start mul/div calculation in advance. The interaction between “membuf” and “mul” includes:

* “mul” gives one number: “mul_this_order”, which indicates the number of the mul instructions. If mul_this_order is 1, it indicates the “mul” module needs the second mul instruction.

* If “membuf” finds one, it will give the “mul” module: mul_vld, mul_para, mul_rs0 and mul_rs1. These signals are necessary to initialize a new mul/div calculation.

* When the “mul” module finishes its new calculation, it will assert: mul_in_vld and the calculation result is in mul_in_data.

* If the result is not accepted by the “membuf” module, it pluses “mul_this_order” 1 to request a new mul/div instruction. if yes, keep “mul_this_order” the same because the last mul instruction is retired with the result.

* If  the “membuf” finds a new one, it continues an iteration; or waits until find a new one.

If the current active instruction is a csr instruction. It will output related signals to the input ports of “sys_csr” module.
When the “membuf” module gets a feedback signal “dmem_resp” from the data bus, it is a good chance to start the next data memory request in the same cycle. It is necessary to get two MEM instructions: one is active_\* signals, which is the current active MEM instruction; the other is alter_\* signals, which is the next of the current.

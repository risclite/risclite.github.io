# Super-Scalar RISCV CPU (SSRV) ------ A synthesizable solution on Super-Scalar and out-of-order

[```Github Repository```](https://github.com/risclite/SuperScalar-RISCV-CPU/)     [```中文版```](https://github.com/risclite/SuperScalar-RISCV-CPU/wiki/%E4%B8%AD%E6%96%87%E5%B8%AE%E5%8A%A9%E7%BB%B4%E5%9F%BA)

## Why super-scalar and Out-of-order ?

These are the main-stream technologies to improve performance of CPU cores. SSRV will provide a vivid example using these two methods on RV32IMC. RISCV is a free and open CPU architecture and it deverses higher performance design than rivials, because more and more people will devote their talent to its development.

SSRV is a synthesizable verilog solution dedicated to super-scalar. SSRV tries to issue N instructions from M instructions gathered. You can assign N and M to have N parallel instructions to be executed, and M instructions to gain more possibility of out-of-order. No matter how many N and M are, SSRV will work smoothly to prove it is a solid design on its theme.

![diagram](/png/diagram.png)

SSRV is based on 4 different multiple-in, multiple-out buffers connected with each other. The central of them is built in “schedule” module, which has “FETCH_LEN” inputs, “EXEC_LEN” outputs and a capacity of “SDBUF_LEN” instructions.

Obviously here M is equal to "SDBUF_LEN" and N is "EXEC_LEN".  

If N and M are given different values, this core will show different Dhrystone Benchmark scores. The next table will list how these key parameters produce different performance cores.

|N--M--N                        |	DHRY(best) |	DMIPS/MHz(best) |	DHRY(legal) |	DMIPS/MHz(legal) |
|-------------------------------|--------------|--------------------|---------------|--------------------|
| 1-- 1-- 1                     |5205	       |2.96                |2645	        |1.51                |
| 1-- 2-- 1 	                |5205	       |2.96	            |2659	        |1.51                |
| 2-- 2-- 2 	                |6366	       |3.62	            |3344	        |1.90                |
| 2-- 3-- 2	                    |6407	       |3.65	            |3471	        |1.98                |
| 2-- 4-- 2	                    |6407	       |3.65	            |3520	        |2.00                |
| 2-- 6-- 2	                    |6407	       |3.65	            |3533	        |2.01                |
| 3-- 3-- 3	                    |6708	       |3.82	            |3689	        |2.10                |
| 3-- 4-- 3	                    |6753	       |3.84	            |3758	        |2.14                |
| 3-- 6-- 3	                    |6799	       |3.87	            |3787	        |2.16                |
| 4-- 4-- 4	                    |6893	       |3.92	            |3758	        |2.14                |
| 4-- 5-- 4	                    |6941	       |3.95	            |3801	        |2.16                |
| 4-- 6-- 4	                    |6941	       |3.95	            |3816	        |2.17                |
| 8--16-- 8	                    |7038	       |4.01	            |3906	        |2.22                |
|16--32--16	                    |7038	       |4.01	            |3921	        |2.23                |

 
These 3 parameters can be random integer. There is a status report when all are assigned to 16. It is obvious that SSRV is a robust solution of out-of-order and super scalar.

    ticks =      261273  instructions =      282644  I/T = 1.081796
          NUM          TICKS       RATIO
            0 --       87638 -- 0.335427 
            1 --      108594 -- 0.415634 
            2 --       37830 -- 0.144791 
            3 --       19693 -- 0.075373 
            4 --        3562 -- 0.013633 
            5 --        1793 -- 0.006863 
            6 --         949 -- 0.003632 
            7 --         566 -- 0.002166 
            8 --         110 -- 0.000421 
            9 --          58 -- 0.000222 
           10 --         360 -- 0.001378 
           11 --          88 -- 0.000337 
           12 --           0 -- 0.000000 
           13 --          12 -- 0.000046 
           14 --           0 -- 0.000000 
           15 --           4 -- 0.000015 
           16 --          16 -- 0.000061 

-------------------------------------

## Have a new perspective on RV32IMC instructions 

SSRV divides RV32IMC instructions into 3 kinds for the purpose of super-scalar and out-of-order:

* SPECIAL instructions: 

The following instructions of them are not assumed to be issued permanently or temporarily, such as unconditional jumps, conditional branch, fence/fencei, or ecall/ebreak. It should be treated as the last one of multiple instructions to be scheduled.

* MEM instructions:   

They are major because they should be handled one by one in-order at every cycle. They includes LSU, CSR and MUL instructions. They should be queued in a buffer and the first-in one is the current active instruction, which has the control of the data bus, or CSR registers. When an exception or interrupt occurs, the current active instruction is responsible for it. 

* ALU instructions:

They are minor because it includes OP or OP-IMM instructions, which are only related with the register: R1~R31. The operation result is queued in another buffer. Only the one,which there is no MEM instruction ahead of, could be permitted to write to the register file. When the current major MEM instruction is retired, a few following ALU instructions gain the pass to the register file.

The detailed illustration is here : [``>>>``](/tutorial/instruction.md)

-------------------------------------

## A Multiple-in, Multiple-out Buffer

The basic component of SSRV is a multiple-in, multiple-out buffer, which has a capacity of multiple elements. FIFO is the simplest one of them. To implement complex operations, you should extend its function and be sure to manipulate it with Verilog. 

![Multiple-in, multiple-out buffer](/png/ProjBuffer.PNG)

It could be detailed here: [``>>>``](tutorial/buffer.md)

-------------------------------------

##  An example on how instructions are managed

A fragment of assembly program is provided to illustrate how the parallel instructions are managed. [``>>>``](tutorial/example.md)

-------------------------------------

### Modules of SSRV

SSRV is based on and inspired by SCR1 of Syntacore. SCR1 has a complete verification and development suite. It provides a solution from C files to BIN files, and systemverilog testbench files loading BIN files to feed simulation of RTL.

SSRV will take over the authority of imem and dmem bus and testbench files do not know whether SCR1 or SSRV is fetching instructions to operate data bus. It is a good idea to have a gold model helpful to debug SSRV. 

SSRV has its own testbench file: /testbench/tb_ssrv.v.  It will run with the testbench file of SCR1 simultaneously. The internal signals of SCR1 are connected to SSRV directly. It can be cancelled by commenting the verilog definition "USE_SSRV" of /rtl/define_para.v. In this case, SCR1 will lead simulation and SSRV is an idle process.

In the testbench file of SSRV, an instance of SSRV's top verilog file is listed here. 

    ssrv_top u_ssrv(
        .clk                    (clk                ),
        .rst                    (rst                ),
        // instruction memory interface	
        .imem_req               (imem_req           ),
        .imem_addr              (imem_addr          ),
        .imem_rdata             (imem_rdata         ),
        .imem_resp              (imem_resp          ),
        .imem_err               (1'b0               ),
        // Data memory interface
        .dmem_req               (dmem_req           ),
        .dmem_cmd               (dmem_cmd           ),
        .dmem_width             (dmem_width         ),
        .dmem_addr              (dmem_addr          ),
        .dmem_wdata             (dmem_wdata         ),
        .dmem_rdata             (dmem_rdata         ),
        .dmem_resp              (dmem_resp          ),
        .dmem_err               (1'b0               )		  
      );
      
For now, SSRV is an instruction processor. It will fetch instructions from instruction memory and follow them to update the register file and operate data bus. It has a data memory interface. Load and store instructions will give different logic value on the data memory signals. If the instruction memory and data memory response well, SSRV will work well, too.

That's the top of SSRV. Let’s review its internal and know how to implement different functions of various modules.

![diagram](/png/diagram.png)
 
* instrman

This module produces PC for fetching instructions. It will start fetching incrementally until the buffer of “instrbits” module is full or a jump occurs. [``>>>``](tutorial/instrman.md)

* instrbits

A preliminary buffer for instructions to store. If its instructions are accepted by the next buffer, it will remove them to save space. [``>>>``](tutorial/instrbits.md)

* schedule

It always takes incoming instructions in until the buffer is full or a SPECIAL instruction appears. It divides the union of incoming instructions and ones stored into two arrays: one is for next stage to be executed; the other is to stay the buffer. [``>>>``](tutorial/schedule.md)

* alu/alu_with_jump

It gives ALU instructions the operation result, or MEM instructions the load/store method, address, and write data. If this ALU is the last one, it will be instantiated by “alu_with_jump”, which has the function of generate jump address for a jump instruction. [``>>>``](tutorial/alu.md)

* mprf

It has a buffer to store the operation result of all ALU instructions. In every cycle, it will write the operation result of instructions, whose preceding MEM instructions all retire, to the register file. [``>>>``](tutorial/mprf.md)

* membuf

Its buffer is to store the load/store method, address and write data. Multiple MEM instructions are queued until the most preceding one have the chance to operate the data bus. If the data bus reports OK, this MEM instruction retires; or arises an exception.[``>>>``](tutorial/membuf.md)

* mul

If a mul instruction is queued in the buffer of “membuf”, it could initialize a mul/div calculation in advance. Its operation result will be queued into a buffer. If this mul instruction becomes the most preceding one, its operation result is fetched from this buffer and dispatched to the register file. [``>>>``](tutorial/mul.md)

* sys_csr

Err, illegal, fencei, fence, sys instructions and csr ones are dealt with in this module. These are related with CSR registers. It could send one of CSR register to the register file for csr instructions; or arise an exception for err, illegal instructions; or initialize a jump operation for fencei, sys instructions. [``>>>``](tutorial/sys_csr.md)

Every module serves one purpose that is to make the “schedule” module dispatches instructions as many as possible. The “instrbits” buffer will accumulate more instructions. The other two buffer will have to clean their reverse to welcome new ones.  


-------------------------------------

## How to parameterize SSRV to balance performance and area

Four multiple-in, multiple-out buffers are embedded in four different modules. Each buffer has its own role to regulate instructions. A multiple-in, multiple-out buffer has 3 parameters: IN_LEN, OUT_LEN and BUF_LEN. To define these parameters of each buffer is a good method to balance performance and area [``>>>``](tutorial/balance.md)

-------------------------------------

## Some tips on simulation and debug

It is a good way to download the github repository and start watching signals concerned [``>>>``](tutorial/simulation.md)

-------------------------------------

## Interrupts and exceptions

How does SSRV deal with interrupts and exceptions ? [``>>>``](tutorial/interrupt.md)



-------------------------------------


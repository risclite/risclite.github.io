
## schedule

-------------------------------------

Please open rtl/schedule.v. This file is the most important Verilog RTL file of SSRV.

The input signals of “instrbits” are a few arrays of “fetch_\*”. At first, two new arrays of “fetch_\*” are added to give each instruction a “para”, which indicates which kinds of instructions it belongs to; a “order”, how many MEM instructions are preceding and not retired. MEM instructions in services are from two module: one is “schedule” module itself, the other is “membuf”, which gathers multiple ones queued.

How many numbers of instructions of “instrbits” are accepted depends on three factors:

* How many numbers are sent from “instrbits”

* How many space is left by the buffer of “schedule”, it is “ `SDBUF_LEN – sdbuf_length”.

* Whether there is one SPECIAL instruction exist.

A logical expression will show these 3 factors: 

    ( fetch_vld & sdbuf_left_vld & fetch_pick_flag[FETCH_LEN-1:0] )

However, if a SPECIAL instruction exists in the buffer of “schedule”, no instruction will be taken in.

To solve the problem of parallel execution, two incremental variables are added: rs_list[i] and rd_list[i]. Every instruction is a function with its one or two inputs: Rs0, Rs1; and one output: Rd. Two or more instructions can be executed together if their registers of Rs or Rd are different. Anyway, there are 31 registers, which can serve several instructions in the same cycle.

Assuming one in-order processor, an instruction with Rs and Rd, is permitted to be executed, which will give following instructions a restriction: their Rs and Rd should not be its Rd. “rs_list” is the restriction register list of Rs; and “rd_list” is the restriction register list of Rd. Any preceding instruction not retired will append its Rs and Rd to these two lists according to whether it is issued to be executed.

If it is issued, its Rs has been referred, and Rs will not be a problem. Only Rd will be appended to “rs_list” and “rd_list”, because the register of Rd is changing and it should not be Rs or Rd of the following instructions. 

If it is hung up, which means its Rs has not been referred, Rd of the following instructions can not be Rs of this stalling instruction. It is necessary to append Rs and Rd of the stalling instruction to “rd_list”, and to append Rd to “rs_list”.

In the main “generate” statements, “hit” is used to indicate whether Rs and Rd of current instruction is appeared in “rs_list” and “rd_list”. “hit” will a key factor to permit the current instruction to be executed. The bellow is some factor to permit the current instruction to be executed:

* hit: Rs and Rd register factor

* mem_not_exec[i]: Whether one MEM instruction is stayed in the buffer. If yes, no other MEM instruction is allowed to be executed, because MEM instructions should be kept in-order strictly.

* (exec_num[i]==\`EXEC_LEN): exec_num[i] is a signal to indicate how many instructions are scheduled. Its total number should not exceed the maximum.

* (mem_num[i]==\`MMBUF_LEN):  The initial number of mem_num[i] is how many instructions are contained in the “membuf” buffer. When one MEM instruction joins the execution list, mem_num[i] pluses one. If the “membuf” buffer is full, no more MEM instruction is permitted.

* (rf_num[i]==\`RFBUF_LEN):  The initial number of rf_num[i] is how many instructions are contained in the “mprf” buffer. When one ALU instruction joins the execution list, rf_num[i] pluses one. If the “mprf” buffer is full, no more ALU instruction is permitted.

Any valid instructions will be evaluated to join “exec_\*” arrays to be executed; or join “sdbuf_\*” arrays to stay in the buffer. Besides that, some miscellaneous signals are generated to help scheduling.

* chain_sdbuf_has_special: the current instruction of staying the buffer is a SPECIAL one, or not.

* chain_sdbuf_mem_num: how many MEM instructions are stayed in the “sdbuf” buffer or in the execution list.

* chain_exec_mem_num: how many MEM instructions are in the execution list.

* chain_exec_rf_num: how many ALU instructions are in the execution list.

* chain_exec_rd_list: the register list of all Rds of MEM instructions in the execution list.

These signals are needed by scheduling. When each candidate is evaluated, we will get two arrays, one is for execution, the other is for staying. These two arrays of signals are written to registers.

The signals: chain_find_mem[i] and chain_find_pc[i] are used to locate the recent MEM instruction. If an interrupt occurs, a major or MEM instruction is needed to be treated as an interrupt starting point. If there is no MEM instruction exist in “membuf” buffer, the one in “schedule” buffer is a perfect candidate. Any preceding ALU instructions will not bring problems when we treat the first MEM instruction of the “schedule” buffer as an interrupt starting point.

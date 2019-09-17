## Simulation and debug

-------------------------------------

It is a good way to download the github repository and start watching signals concerned. The structure of the github repository is listed here:

* rtl/: Synthesizable RTL code of SSRV. Most of .v files have been elaborated.
	
* testbench/: Only a Verilog testbench file: tb_ssrv.v. It instantiates the top file of SSRV. It connects signals of SCR1 to ports of this top module.

* scr1/: The simulation package of SCR1. 

    * src/: The source file of RTL and testbench provided by SCR1. All is the format systemverilog. SSRV only has one testbench file: /src/tb/scr1_top_tb_ahb.sv modified to adopt SSRV as one design under test.
	
    * build/: It includes .elf, .hex and .dump files for test cases built by compilers. The testbench file of SCR1 will load .hex files listed by /build/test_info. You can comment some lines of /build/test_info with “#” to forbid simulation on these cases. If you need know the meaning of instructions, please open .dump file of this case.
	
    * sim/: The directory is a place where you can start simulation. There are two .do file for your reference. One is compile.do, which lists include directories and RTL/testbench files of both SCR1 and SSRV. The other is sim.do, which treats the testbench files of both SCR1 and SSRV as two concurrent entities.You should modify these two files manually unless your simulation tool is Modelsim.

When you have finished downloading, just enter the directory: /scr1/sim/ and run the “compile.do” file to compile all related files into one library. Then, run the “sim.do” to start simulation. If you enter some command like this: run -all, you can see the transcript displaying:

    ---Begin testing: ../build/coremark_imc.hex 
    CoreMark 1.0
    ticks =      270962  instructions =      282644  I/T = 1.043113
               0 --       86936 -- 0.320842 
               1 --      116807 -- 0.431083 
               2 --       41456 -- 0.152996 
               3 --       20127 -- 0.074280 
               4 --        5636 -- 0.020800 
    Jump number is       32231 --ratio: 0.118950
    MEM number is       71757 --ratio: 0.264823
    2K performance run parameters for coremark.
    CoreMark Size    : 666
    Total ticks      : 2709
    Total time (secs): 0
    ERROR! Must execute for at least 10 secs for a valid result!
    Iterations       : 1
    Compiler version : GCC8.3.0
    Compiler flags   : -O2 -funroll-loops -fpeel-loops -fgcse-sm -fgcse-las
    Memory location  : STATIC
    seedcrc          : 0xe9f5
    [0]crclist       : 0xe714
    [0]crcmatrix     : 0x1fd7
    [0]crcstate      : 0x8e3a
    [0]crcfinal      : 0xe714
    Errors detected
    ---../build/coremark_imc.hex Test PASS


I have added some program codes of statistics, which will count how many instructions and ticks it takes from the timer being pressed to being shut. We could know instructions per cycle. Since SSRV is super-scalar, it will list the ticks of different parallel instructions and their ratio of all ticks it takes. You can use that to adjust your parameters.

If you do not need this, you can comment “BENCHMARK_LOG” of /rtl/define_para.v to eliminate it.

You can add stalling cycles to instruction or data memory response. Please open scr1/src/tb/ scr1_top_tb_ahb.sv, find the below statements:

    imem_req_ack_stall = 32'hffffffff;
    dmem_req_ack_stall = 32'hffffffff;

SCR1 allows you to modify acknowledge stalling cycles. You can give “imem_req_ack_stall” more “0” bits to add stalling cycles for instruction memory. “dmem_req_ack_stall” is for data memory.

If you want a golden model, SCR1 is a ideal one, which you can benefit from commenting “USE_SSRV” of /rtl/define_para.v. The DUT will be switched from SSRV to SCR1. The below signals of SCR1 are helpful: 

    /scr1_top_tb_ahb/i_top/i_core_top/i_pipe_top/i_pipe_exu/curr_pc
    /scr1_top_tb_ahb/i_top/i_core_top/i_pipe_top/i_pipe_exu/new_pc_req
    /scr1_top_tb_ahb/i_top/i_core_top/i_pipe_top/i_pipe_exu/next_pc
    /scr1_top_tb_ahb/i_top/i_core_top/i_pipe_top/i_pipe_mprf/mprf_int

“curr_pc” is PC of the current instruction; “new_pc_req” is a flag of jumping operation; “next_pc” is the target address; mprf_int is the register file: R1~R31.


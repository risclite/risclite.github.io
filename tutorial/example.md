## An example on how instructions are managed

-------------------------------------

![diagram](/png/diagram.png)

Let's take an example to discuss how hinstructions are managed. 

    138e:	01059313          	slli	t1,a1,0x10
    1392:	01035f93          	srli	t6,t1,0x10
    1396:	01efc333          	xor	t1,t6,t5
    139a:	030e                	slli	t1,t1,0x3
    139c:	07837313          	andi	t1,t1,120
    13a0:	007fff93          	andi	t6,t6,7
    13a4:	c398                	sw	a4,0(a5)
    13a6:	01f36733          	or	a4,t1,t6
    13aa:	c11c                	sw	a5,0(a0)
    13ac:	00871313          	slli	t1,a4,0x8
    13b0:	c3d4                	sw	a3,4(a5)
    13b2:	00e36fb3          	or	t6,t1,a4
    13b6:	01f69023          	sh	t6,0(a3)
    13ba:	01d69123          	sh	t4,2(a3)
    13be:	873e                	mv	a4,a5
    13c0:	869e                	mv	a3,t2
    13c2:	8796                	mv	a5,t0

In the first stage, "instrman" module will initiate a request to "instruction memory", which has a PC address : 13b0. Since it could fetch 4*32 bits, hex data : "873e_01d69123_01f69023_00e36fb3_c3d4" will appear on the imem bus "imm_rdata" in the next cycle, which is the instructions of addresses: "13be+13ba+13b6+13b2+13b0".

In the second stage, a multiple-in, multiple-out buffer in the module "instrbits" will welcome its incoming data: imem_rdata "873e_01d69123_01f69023_00e36fb3_c3d4". This buffer has stored two instructions: "00871313_c11c", whose addresses are 13ac and 13aa. These two  clusters of hex data will join together: 873e_01d69123_01f69023_00e36fb3_c3d4_00871313_c11c. From the lowest bit, "FETCH_LEN",which is 4 here,  instructions will be gathered to output to the next module. Now, we know the addresses of the 4 instructions are 13b2, 13b0, 13ac, 13aa.

The multple-in, multiple-out buffer of the module "schedule" has 4 incoming new instrctions: 13b2, 13b0, 13ac, 13aa. Since the buffer has stored 3 instructions: 13a6, 139c, 139a and its capacity is "SDBUF_LEN", which is 6 here, it will have to accept 3 of 4.
These 6 instructions are candidates:

    139a:	030e                	slli	t1,t1,0x3
    139c:	07837313          	andi	t1,t1,120
    13a6:	01f36733          	or	a4,t1,t6
    13aa:	c11c                	sw	a5,0(a0)
    13ac:	00871313          	slli	t1,a4,0x8
    13b0:	c3d4                	sw	a3,4(a5)

We will choose "EXEC_LEN",which is 4 here, instructions between them. Since these is always data dependency of instructions, the instructions : 139a, 13aa and 13b0 are winers.The losers will stay in the buffer to wait for the next try. These 3 winers are going to registers, which means the end of the second stage.

    139a:	030e                	slli	t1,t1,0x3
    13aa:	c11c                	sw	a5,0(a0)
    13b0:	c3d4                	sw	a3,4(a5)

In the third stage,these 3 instructions will pass through their own ALU module. The instruction 139a will have a destination register: t1 and its new data. The othes are different because they need operations on data bus, not just a destination register and a simple new data. 

The buffer in the module "membuf" will have two newcomers: 13aa and 13b0, which are queued to operate the data bus one by one. The buffer has accepted one instrction : 13a4 in the last cycle. It is operating the data bus now. The newcomers have to be queued in the buffer.

    13a4:	c398                	sw	a4,0(a5)
    
The instruction 139a will have a different destination: mprf module. The inner buffer will accommodate it and make them be queued after two early instructions: 13a0 and 1396.

    1396:	01efc333          	xor	t1,t6,t5
    13a0:	007fff93          	andi	t6,t6,7
    
These 3 instructions will not write to the register file: R1~R31 directly. The instruction 13a4 is operating the data bus now, and we do not know whether it will be successful. Every ALU instructions queued here should be checked whether it is ahead of the address 13a4. Only instructions ahead of it are allowed to write to the register file. The instructions following it will have to wait for the acknowledgement of data bus, and then they have a new chance to check whether its address is ahead of the new MEM instruction operating the data bus.

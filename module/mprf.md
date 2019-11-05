
## mprf

-------------------------------------

This module manages the register file. It includes reading from the register file by “alu” modules; writing to the register file by “alu” modules and “membuf” module.

It has a buffer to contain writing data from the “alu” modules.  The “membuf” module will send “mem_release” signal to indicate that a MEM instruction has been retired successfully. Orders of all ALU instructions stored in this buffer will decrease 1 until it reaches 0. After that, only ALU instructions whose order is zero have possibility to write to the register file.

The writing from “membuf” module happens directly because it is from a major MEM instruction. These two signals are mem_sel and mem_data.

Reading from the register file includes two steps. The first step is to look up from the register file, like searching a word from a dictionary. The next step is to check whether this register is overridden by one element of the buffer, like searching an item from an erratum.

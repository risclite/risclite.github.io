## sys_csr

-------------------------------------

This module is related with CSR and sytem control. Any CPU core which is declared to support RV32IMC has the common function implemented by SSRV’s modules except this one. You can customize your different core through rewriting this module and keeping other modules unmodified. Other modules will bring you the high-performance and flexible attribute. Rewriting this module will give you flexibility to adopt different application mode.

This module will response with incoming system or csr instruction. If it is a csr instruction, it will write Rs to its CSR and output CSR to “csr_data”. The “membuf” module will copy “csr_data” to the register file.

If it is a system instruction, it can initialize a jump operation to the target address, which is provided by one of CSR registers.
In this module, only 7 CSR registers are implemented. They are the minimal solution to adopt with the simulation environment of SCR1. Only 3 system instruction are described to jump a target address. That is also a minimal solution.
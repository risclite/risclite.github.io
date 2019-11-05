## mul

-------------------------------------

This module will implement mul or div operation. It costs cycles which depends on how many bits of the multiplier or quotient. It is an iteration on shift and add/subtract operations until each bit of the multiplier is dealt with or the shrinking dividend is less than the divisor.

“calc_start” is a signal to start the calculation of mul/div. If one of the multipliers is zero, the divisor is zero or the dividend is less than the divisor, the calculation will never be started and “write_start” is asserted to initialize writing a buffer of the result.

If the buffer is full, it will never start a new calculation until the “membuf” module retires the current active mul instruction through copying the mul/div result to the register file.

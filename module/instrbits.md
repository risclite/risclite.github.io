## instrbits

-------------------------------------

Please open rtl/instrbits.v.

It has a preliminary buffer to store “imem_rdata” and provides “FETCH_LEN” number of instructions from the buffer.

    output `N(`FETCH_LEN)              fetch_vld,
    output `N(`FETCH_LEN*`XLEN)        fetch_instr,
    output `N(`FETCH_LEN*`XLEN)        fetch_pc,
    output `N(`FETCH_LEN)              fetch_err,
    input  `N(`FETCH_OFF)              fetch_offset

The output signals of fetch_\* are arrays of different information. “vld” is for valid signals of every element; “instr” is for an instruction; “pc” is for a PC address; “err” is for error indication. When this module sends fetch_* to the next module, it expects a return signal: “fetch_offset”, which means how many instructions are taken in and it is safe to eliminate them.

If a jump signal fires a fetching request, the jump target address is not always aligned. SSRV support multiple-word fetching style and it is necessary to remove redundant misaligned half-words from “imem_rdata”.

After that, incoming half-words and half-words stored join together. After removing “fetch_offset” number of instructions, the reset of them will stay in the buffer.

The union of half-words incoming and stored will be analyzed to form “FETCH_LEN” number of instructions.

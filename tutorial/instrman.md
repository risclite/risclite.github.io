## instrman

-------------------------------------


Please open rtl/instrman.v and review it, the next lists are helpful.

* Manage different jump-starting signals, form one integration set of jump signals: jump_vld, jump_pc.

* “buffer_free” is a signal that means the preliminary buffer is free to accept new coming instructions. This module will fire fetching operation until it turns to low level or a jump signal asserts.

* “req_sent” is used to indicate a fetching request has been sent. If there is no fetching request started, a new fetching request could be initialized in any cycle; if a fetching request has been sent, the next fetching request will have to wait for the acknowledgement of “imem_resp” and after that, the new fetching request can be started. A logic expression: ~req_sent & imem_resp is used to express when to start a new fetching request.

* “line_requested” is a signal to indicate “imem_rdata” is a valid cluster of instructions. Only when a fetching request has been sent and “imem_resp” is OK, “imem_rdata” is valid to be forwarded to the buffer.

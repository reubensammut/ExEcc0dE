-----
title: Assignment 3 - Egg Hunter
author: Reuben Sammut
-----

> This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification: [http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)
> 
> Student ID: SLAE-510

###Requirements
The third assignments requires the study of the egg hunter shellcode. After the study, a working demo should be created to show the egg hunter working. One should also be able to configure the Egg hunter for different shellcodes.

###What is the egg hunter shellcode?
Looking online for the egg hunter shellcode, all seems to point to a paper [Safely Searching Process Virtual Address Space](http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf) by Matt Miller (skape). The paper starts off with the realistic problem where an exploit vector, such as the place where a buffer overflow can occur, might be too small for our shellcode. 

####Assumptions
With a very small buffer we can exploit, there is very little we can do. So this paper makes the assumption that we have a larger payload that we can somehow access. From this assumption we can build up a small payload that will fit into the buffer whose job will be to look for the larger payload. 

This method is not without its problems. With the larger payload somewhere in the VAS(Virtual Address Space) of the process, we have no other way than to do a linear search through the address space. The problem this poses is that the linear search will undoubtedly hit regions of unallocated or unaccessible memory. Dereferencing such regions would cause what the paper calls "Bad Things", which usually refer to segmentation faults. So Miller goes to address different techniques to avoid failing exploits due to these "Bad Things".

####Properties
The paper talks about three properties that the egg hunter should have. 

1. **Robust** - As already discussed, the egg hunter shellcode should be able to look for the large payload in all the memory without failing on invalid regions. 

2. **Small** - The reason why we need an egg hunter shellcode in the first place is that our target shellcode does not fit in the buffer we can exploit. So we need the egg hunter shellcode to be as small as possible. 

3. **Fast** - While speed is not theoretically required for the egg hunter shellcode to work, it would be unfeasible to wait a long time for the egg hunter to find the egg. This property should also not violate the previous two properties.



####Extra challenge




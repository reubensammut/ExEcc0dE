-----
title: Assignment 2 - Shell reverse tcp
author: Reuben Sammut
-----

> This blog post has been created for completing the requirements of the SecurityTube Linux Assembly Expert certification: [http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/](http://securitytube-training.com/online-courses/securitytube-linux-assembly-expert/)
>
> Student ID: SLAE-510

###Requirements

The second assignment requires a shellcode which connects to a configured IP and port, and executes a shell on a successful connection. Both the IP and port should also be easily configured.

###Implementation

This assignment is very similar to the first assignment. Whereas in the first assignment it was required for the shellcode to be able to listen to a port, in this assignment, the shellcode should do the opposite. This time, the shellcode should connect to a configured IP and port. A case for this assignment is that it is often the case where the machine on which the shellcode is to be deployed, is usually behind a router and has no direct connection to the internet. Instead of finding a way to enter the router to configure port forwarding, it is best to make the shellcode connect to an outside machine which the attacker has control over.

Unlike the case for the first assignment, where maintaining shell access through the connection is a plus, in the case of a reverse TCP shellcode, it does not make sense to have it constantly trying to connect. As such, we do not need a complex setup where we fork on a successfully established connection. However, more for the sake of completeness of the shellcode, we have to take care of handling failed connections. This means that if connect fails for any reason, we should not try to redirect the streams or spawn a shell. If we actually try to redirect the streams on a failed connection, the application will actually throw a segmentation fault.

For this assignment, the list of system calls is shorter than the one in the first assignment. Below is a list of the system calls required:

- **socket** – create socket endpoint
- **connect** – initiate connection on socket
- **dup2** – duplicate file descriptor(stream redirection)
- **execve** – execute program, as usual we’ll use a shell
- **write** – write a message to a file descriptor(for error message)
- **exit** – exit with a status code

The socket calls, in this case socket and connect are done as per assignment 1, that is using the socketcall system call. We could use the same table of `#defines` found in `/usr/include/linux/net.h`. For a TCP client, after we create the socket, we call connect (using the socketcall system call) as can be seen in the below diagram.

<div class="figure">
<a href="/images/slae_02-01_connect_socketcall.png">
<img src="/images/slae_02-01_connect_socketcall.png" />
</a>
Setting up the socketcall system call for connect
</div>

One of the requirements for this assignment is to create a shell if the connection has succeeded. This means that we have to check the return code of socketcall. The man page for connect states that connect returns 0 on success and -1 for failure, with _errno_ set appropriately. In assembly both the return code and error are returned in $eax. Here we do not care what the error was. We are just interested in whether the connection succeeded or not. Unfortunately, the Zero Flag is set both when the connection succeeds and when it fails. To circumvent this, we have to check whether eax is set to 0 by using `test eax, eax`. Test works by performing the bitwise AND operation on the two operands. If the two operands are equal, the Zero Flag is set if and only if both operands are 0. So by doing an AND of $eax with itself, the Zero Flag will be set if $eax is 0. From this we can decide whether we can continue with the shell launch or close the application.

If the connection has succeeded, we would do the same thing as we did after creating the child in the first assignment. We close all file descriptors and reopen them on the socket endpoint using dup2. After this has been done for standard input, output and error, we can spawn the shell using execve.

In the case when the connection has failed, I decided to print an error message and exit. In reality, you would not want the application to exit, but for the sake of this assignment and simplicity, this seemed to be the most viable way to implement this. Printing is done using the write system call and after it is done, the shellcode calls the exit system call.

###Testing

Failure case:
<div class="figure">
<a href="/images/slae_02-02_connect_failed.png">
<img src="/images/slae_02-02_connect_failed.png" />
</a>
Error message on connection not established
</div>

<div class="figure">
<a href="/images/slae_02-03_run_shellcode.png">
<img src="/images/slae_02-03_run_shellcode.png" />
</a>
A connected reverse tcp shell
</div>

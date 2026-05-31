## linux-architecture-notes

## The core components of Linux (kernel, user space, init/systemd)
1.kernel – kernel contains the  code base of linux. It is the heart of the operating system. It is a brideg between software and hardware devices.
2. init/systemd – it is the first process started by kernel. It is PID 1. It starts all the other remaining process.

3. user namespace – it is an isolated environment created by kernel for each user  that isolates users and group Id’s in a process.

## How processes are created and managed
Process are managed and created by kernel.

## What systemd does and why it matters.
It is the first program the kernel launches on boot. It initializes the system, tracks resources, and acts as a master service manager. It matter because earlier there was traditional, slower startup systems (like SysVinit) which took a lot of time, was not secure and did not provide reliability.

## Explain process states (running, sleeping, zombie, etc.)
1.	Running (R) – the process is either running r waiting in queue for CPU space.
2.	Interruptible Sleep (s) – the process is waiting for an event like user input or network signal.
3.	Uninterruptible Sleep (d) – Usually seen during critical I/O operations(like reading disk). It is will not respond to any signal like (s).
4.	Stopped (t) – the process has been suspended.
5.	Zombie (z) -  the process which has been completed but still has an entry in the process table. Exits after it parent process is completed.


## List 5 commands you would use daily
1.	Cat
2.	Ls
3.	Pwd
4.	Touch or echo
5.	Mkdir or rmdir



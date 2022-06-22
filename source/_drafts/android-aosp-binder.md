---
title: Android Binder
tags: Android
-------------

一、Binder是什么

<center>
    <img src="../images/android-binder-what.svg" width="50%"/>
</center>

+ An IPC/component system for developing object-oriented OS services
 - Not yet another object-oriented kernel
 - Instead an object-oriented operating system environment that works on traditional kernels, like Linux!

+ Essential to Android!
+ Comes from OpenBinder
 - Started at Be, Inc. as a key part of the "next generation BeOS" (~ 2001)
 - Acquired by PalmSource
 - First implementation used in Palm Cobalt (micro-kernel based OS)
 - Palm switched to Linux, so Binder ported to Linux, open-sourced (~ 2005)
 - Google hired Dianne Hackborn, a key OpenBinder engineer, to join the Android team
 - Used as-is for the initial bring-up of Android, but then completely rewritten (~ 2008)
 - OpenBinder no longer maintained - long live Binder!

+ Focused on scalability, stability, flexibility, low-latency/overhead, easy programming model

二、IPC

+ Inter-process communication (IPC) is a framework for the exchange of signals and data across multiple processes
+ Used for message passing, synchronization, shared memory, and remote procedure calls (RPC)
+ Enables information sharing, computational speedup, modularity, convenience, privilege separation, data isolation, stability
 - Each process has its own (sandboxed) address space, typically running under a unique system ID
+ Many IPC options
 - Files (including memory mapped)
 - Signals
 - Sockets (UNIX domain, TCP/IP)
 - Pipes (including named pipes)
 - Semaphores
 - Shared memory
 - Message passing (including queues, message bus)
 - Intents, ContentProviders, Messenger
 - Binder!


三、为什么使用Binder

+ Android apps and system services run in separate processes for security, stability, and memory management reasons, but they need to communicate and share data!
 - Security: each process is sandboxed and run under a distinct system identity
 - Stability: if a process misbehaves (e.g. crashes), it does not affect any other processes
 - Memory management: "unneeded" processes are removed to free resources (mainly memory) for new ones
 - In fact, a single Android app can have its components run in separate processes
+ IPC to the rescue
 - But we need to avoid overhead of traditional IPC and avoid denial of service issues
+ Android’s libc (a.k.a. bionic) does not support System V IPCs,
 - No SysV semaphores, shared memory segments, message queues, etc.
 - System V IPC is prone to kernel resource leakage, when a process "forgets" to release shared IPC resources upon termination
 - Buggy, malicious code, or a well-behaved app that is low-memory SIGKILL'ed
+ Binder to the rescue!
 - Its built-in reference-counting of "object" references plus death-notification mechanism make it suitable for "hostile" environments (where lowmemorykiller roams)
 - When a binder service is no longer referenced by any clients, its owner is automatically notified that it can dispose of it

+ Many other features:
 - "Thread migration" - like programming model:
    Automatic management of thread-pools
    Methods on remote objects can be invoked as if they were local - the thread appears to "jump" to the other process
    Synchronous and asynchronous (oneway) invocation model
 - Identifying senders to receivers (via UID/PID) - important for security reasons
 - Unique object-mapping across process boundaries
    A reference to a remote object can be passed to yet another process and can be used as an identifying token

 - Ability to send file descriptors across process boundaries
 - Simple Android Interface Definition Language (AIDL)
 - Built-in support for marshalling many common data-types
 - Simplified transaction invocation model via auto-generated proxies and stubs (Java-only)
 - Recursion across processes - i.e. behaves the same as recursion semantics when calling methods on local objects
 - Local execution mode (no IPC/data marshalling) if the client and the service happen to be in the same process
+ But:
No support for RPC (local-only)
Client-service message-based communication - not well-suited for streaming
Not defined by POSIX or any other standard
+ Most apps and core system services depend on Binder
 - Most app component life-cycle call-backs (e.g. onResume(), onDestory(), etc.) are invoked by ActivityManagerService via binder
 - Turn off binder, and the entire system grinds to a halt (no display, no audio, no input, no sensors, …)
 - Unix domain sockets used in some cases (e.g. RILD)











<center>
    <img src="../images/android-basic-aidl-overview.png" width="50%"/>
</center>


[1.Deep Dive into Android IPC/Binder Framework](https://www.protechtraining.com/static/slides/Deep_Dive_Into_Binder_Presentation.html#title-slide)
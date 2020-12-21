# System Calls and Kernels
System calls can be
### Blocking:
Here the caller user process has to wait while the sys call is busy and not yet finished.
### Non Blocking:
System call returns quickly, but the user process has to make extra sys calls to check if the results are available.

### Other info:
Arguments to sys calls will be pointers or tiny variables as actual data is copied to/from user space.

### Making system calls with bypassing the c-library
~~~
 #include <sys/syscall.h>

int main(int argc, char** argv) {
char message[20] = "Hello World!\n";
// Invokes the 'write' system call that writes
// some bytes into the standard output.
/*
   R_write: is syscall number.
         1: file descriptor fd,in this case std output
   message: user space buffer pointer
        13: is length
*/

syscall(__NR_write, 1, message, 13);
return 0;
}
~~~

### How sys call works
Before making the syscall the registers of the CPU are filled with appropriate sys call number, other arguments. When syscall is now made, an interrupt is generated and resulting in interrupt code in kernel.
Sys call Interrupt code picks up the sys call number and other arguments. Finally it searches **system call table** to find appropriate function to be invoked in kernel.
Thus user space is no longer using the CPU, but its the kernel code which is fetched by the CPU after the syscall and we are in kernel mode as CPU switches the mode to kernel.

### Tool to check sys call trace
Sys call trace can be done with strace tool as:
$ strace < app name which has sys call >

printk() messages in kernel can be seen with
$ dmesg

### Good practice
When a new system call needs to be implemented its a good practice to implement the kernel part as a module instead of making kernel changes directly to avoid frequent reboots and crash.

## Monolithic vs Microkernel
Monolithic kernel is made up of single process containing all the services e.g. memory managment, device drivers, file system etc.
A small bug in the monolithic kernel e.g. in a driver will cause a system crash.
More time needed to compile and load a kernel.

Microkernel have separate processes for each service.
These kernel services such as memory management, file system etc run as user space.These processes communicate with IPC.
A small bug will only result in the service crash and can be restarted (e.g. driver can be restarted).
Compile time is low as only changes done in the service can be compiled not necessarily the whole kernel.

### Ways to communicate with Linux kernel modules

1. /dev: device files are available here and you can read and write to these files.
2. /procfs: Files here can be used to communicate with kernel module
3. /sysfs: Another filesystem in linux and is a new version of procfs.

### Enter and exit of kernel modules:

We need to register init and exit callbacks with the kernel.

module_init(hwkm_init);
module_exit(hkwm_exit);

These functions (i.e. hwkm_init() and hwkm_exit()) get called when the module is loaded and unloaded respectively.
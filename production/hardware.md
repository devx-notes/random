# Calculating Hardware Needs

## 1. CPU Cores 

Cores are the fundamental processing units of a CPU. Each core can execute instructions independently. Having more cores allows a CPU to perform more tasks at the same time, leading to better multitasking and performance, especially for tasks that can be divided into smaller, parallel sub-tasks. 

### CPU Threads

Threads are sequences of instructions that a CPU core can execute. In the context of a CPU, threads can be thought of as virtual cores.

- Simultaneous multithreading (SMT, also known as Hyper-Threading), allows a single physical core to handle multiple threads at once by switching between them rapidly. 
- This improves resource utilization and overall performance, especially in applications designed to take advantage of multiple threads. 

> [!NOTE]
> 8 Cores, 12 Threads means the CPU has eight physical processing units. While each of the eight cores can handle one task, they can also manage an additional virtual thread, resulting in a total of twelve active instruction streams. 

## 2. Calculating RPS

## 3. Memory

A 32-bit systems have a theoretical RAM limit of 4GB. The number of unique addresses is given by $2^{32}$. Since each address typically points to $1$ byte, the total addressable memory is $4,294,967,296$ bytes (4 GB). 

> [!NOTE]
> In a 32-bit CPU, the number 32 refers to the width of the data bus, address bus, and the size of registers within the processor. This means the CPU can process and transmit 32 bits of data at a time, and also access  $2^{32}$ memory addresses. 

## References

- [Medium | How to Calculate Server Max Requests per Second](https://medium.com/geekculture/how-to-calculate-server-max-requests-per-second-38a39bb96a85)
- [Oracle | Determining Hardware Capacity Requirements](https://docs.oracle.com/cd/E13222_01/wls/docs81/capplan/capgen.html)
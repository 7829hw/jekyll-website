---
title: "The Student Supercomputer Challenge Guide Chapter 1: Development and Application of Supercomputing"
author: khw
date: 2026-03-05
categories: [Study, Supercomputing]
tags: [LAB]
pin: true
math: true
mermaid: true
---

## Chapter 1 Development and Application of Supercomputing

## 1.1 Why Do We Study Supercomputing?

A supercomputer is a computer system that provides significantly higher computing power than normal personal computers. To provide a high computing performance, supercomputers are generally in the form of highly parallel systems. While supercomputers were mainly used by national labs and national defense agencies when they first came out, more and more industries are starting to use supercomputers for their research and development.

The increasing demand of computing power of different applications promotes the continuous development of supercomputers, and the requirements of different applications are quite distinct. For instance, numerical atmospheric simulation requires tight timelines; nuclear simulation requires high accuracy; intensive transaction processing requires high throughput; and some other applications might have demands on all three aspects of speed, accuracy, and throughput. These different challenges from different applications have been a major driving force for the development of supercomputing technologies.

In most cases, the demand for computing power in supercomputing applications is continuing to grow, and in general beyond the development of current supercomputer systems. Typical examples include numerical simulations in science and engineering. For these simulation problems we sometimes need to repeat the computation for a large number of times to get useful results. Moreover, users expect all of these simulation steps to be finished within an affordable timespan.

For example, in the manufacturing industry, a simulation should be finished within a few seconds or minutes if possible. During the design process, a simulation cycle that takes weeks is not acceptable. Only with an acceptable simulation time cycle, can the designers work in an efficient way. Moreover, when the simulation system becomes more complicated more time is needed to finish the simulation. Some applications have speci fic requirements on the time to solution. The most typical example is numerical weather prediction (NWP). It is meaningless if we spend two days completing one day's forecast, no matter how accurate it is. Therefore, the problems where we are not able to compute solutions within affordable time become big challenges.

Again, we can take NWP as an example. NWP is a widely used application that requires supercomputers. The modeling of the atmosphere is generally performed by discretizing the atmosphere into 3D grids. We then apply complex physics equations to advance in time. The variables (temperature, pressure, humidity, wind, etc.) within each grid are generally computed based on the values of the previous time step. To perform a meaningful weather simulation, we need to perform the computation within each grid for a large number of time steps, while the same computation also needs to be performed for all the grids in the 3D model. These two aspects add up to a huge quantity of computation that we need to perform.

To provide the weather forecast for several days, we need to have a large modeling domain, as the atmosphere could be affected by long-distance events. Assume we divide the entire atmosphere into grids in the size of "1 mile × 1 mile × 1 mile" , and we simulate a layer of 10 miles (10 girds at the vertical axis), we need around $5 \times 10^8$ units to simulate the entire area. Assume that we need to perform 200 floating-point operations for each grid at each time step, then we need to perform $10^{11}$ floating-point operations to finish the simulation of one time step. Assume that we want to simulate 7 days using the time step of 1 min, we need to simulate for $10^{4}$ time steps, which amounts to $10^{15}$ floating-point operations. For a 1 Gflops (giga-flops, $10^9$ floating-point operations per second) computer, such a simulation would take $10^6$ s (over 10 days). If we need to complete this simulation within 5 min, a 3.4 Tflops (tera-flops, $10^{12}$ floating-point operations per second) supercomputer is needed. Moreover, if we want to finish the simulation within 5 min with a higher resolution of grids (0.1 mile × 0.1 mile × 0.1 mile), the total number of floating-point operations increases to $10^{19}$ (note that the decrease of grid size would usually lead to a smaller time step), and we would need to have a 34 Pflops (peta-flops, $10^{15}$ floating-point operations per second) supercomputer (as a reference, the peak performance of Tianhe-2 supercomputer is 33.86 Pflops).

Predicting the movements of objects in the aerospace is another challenging problem that requires a huge amount of computing power. The objects attract each other due to gravitation law. To provide a movement prediction of each object, we need to fi nd out all of the forces that affect it, and make a vector addition to get the composition. For a system with $N$ objects, each object takes $N - 1$ forces, adding up to $N^2$ computations roughly. After determining the forces and the new locations of objects, we need to repeat the computation for the next time step. Take the galaxy as an example. There are around $10^{11}$ stars in total. Therefore, we need to perform $10^{22}$ computations for each time step. Even if we use a highly efficient approximation algorithm with the complexity of $N \log( N )$, the total amount of computation is still enormous (around $10^{12}$ ), which would take an extremely long time to finish on one computer node. Assume that each computation takes only one micro second (this is an optimistic assumption, as each computation involves a large amount of addition and multiplication operations), when using the $N^{2}$ algorithm, one iteration would take $10^{9}$ years to finish. When using the $N \log( N )$ algorithm, it can still take one year to finish one iteration.

The two examples we mentioned above (weather forecast and sky simulation) are the applications in the traditional scientific computing domain. People will continue to come up with more complex applications (such as virtual reality) that demand more and more computing power. In conclusion, no matter how fast the supercomputer is, there will always be new applications that require more.

In 1992, the US government initiated a 5-year plan of high performance computing and telecommunication. In that plan, for the first time, they proposed 12 urgent and challenging problems to solve, including fluid simulation, atmospheric modeling, fluid turbulence, pollution dispersion, human genome, ocean circulation, drug design, dyeing quantum dynamics, semiconductor modeling, supercomputer modeling, combustion systems, vision, and cognition. For some of problems, such as the human genome plan, we have already made meaningful progress; while for some other problems, we are still putting more efforts. In the twenty-first century, the following problems are still considered the most challenging: climate modeling and prediction; design improvements for automobiles, airplanes, and ships (related to fluid dynamics, fuel consumption, structural design and collision avoidance, etc.); bioinformatics; health and safety of the society; earthquake prediction; geophysics exploration; astrophysics (simulation of celestial evolution); materials science and nanotechnology; organization research (behavior simulation and counter-terrorism); nuclear simulation; numerical wind tunnel; and pollution detection and prevention.

In a supercomputer system, employing multiple processors to solve one problem is one common way to improve the computing speed, which has been studied for years. In such a method, the solving process is divided into several parts that could be executed in parallel, and each processor takes charge of one part. This computing method is called parallel computing. The computing platform, which is a parallel machine, could be a computer node that contains multiple customized processors, or a system that consists of multiple independent computers connected together.

In general, we expect to achieve significant speedup when taking the approach of parallel computing. In an ideal case, we expect $N$ times speedup when using $N$ computer nodes. However, in practical cases, it is difficult to decompose a problem into totally independent sub-tasks. The decomposed components usually need to exchange data with each other. Therefore, the actual speedup we can achieve is highly dependent on the parallelism within the algorithm.

Besides accelerating the solving process, the parallel computing approach can also enable the solution for large problems or more accurate results within an acceptable time frame. For example, when simulating a lot of physics phenomena, we discretize the problem domain into 2D or 3D grids. When using parallel computing with multiple computer nodes, we can process more grids within the same amount of time, thus leading to a more accurate depiction of the problem. Another factor is that a multi-node system would provide a significantly larger memory space than a single node. Therefore, it is possible to fi t much larger problems.

![Image](/assets/img/posts/2026-03-05-The_Student_Supercomputer_Challenge_Guide_Chapter_1_artifacts\image_000001_fe5f08e87dcf5948270a0967dd3f2aed60cac4f357ad31546c073362f293f695.png)

Fig. 1.1 Differences between classic science and contemporary (modern) science

After describing the urgent demands for supercomputers, now we will turn to the discussion about the main purposes of supercomputer research. In general, the main purposes of supercomputer research are as follows:

Firstly, computation has become the third method of exploring the world and making new discoveries, along with the scientific theory and experiments. Moreover, as time goes by, the computational approach is becoming more and more important.

The difference between classic science and contemporary science is shown in Fig. 1.1. Classic science is based on observation, theory, hypothesis and experimentation. In contrast, contemporary (modern) science adds numerical simulation to the process. By using supercomputers, we can use numerical simulation to verify our theory and hypothesis, instead of the traditional experiment approach. In many cases, the numerical simulation approach can be much more efficient.

Secondly, the development of supercomputers is a strong driver for the development of related computer science and technologies, especially for parallel architectures, high-bandwidth and large-capacity memory hierarchies, high-bandwidth and low-latency network, parallel programming methods, and so on.

Thirdly, research on supercomputers promotes the development of a number of inter-disciplinary subjects, such as computational mathematics, computational physics, bioinformatics, computational economics, computational electromagnetics, and computational geophysics.

Finally, the development of supercomputers is a good demonstration of the country's economic and technology power.

In order to reflect and promote the development of supercomputers in the world, from 1993, the TOP500 list was announced twice a year, at the supercomputing conferences in Germany and the USA respectively. The TOP500 list includes the 500 fastest computer systems in the world, which are ranked by their LINPACK (the most widely accepted floating-point performance benchmark) performance. For a long time, the supercomputers in the USA dominated the top positions of the TOP500 list.

Coming into the twenty- first century, supercomputer technologies in Japan have developed quickly. The "Earth Simulator" supercomputer in Japan provided a peak performance of 35.6 Tflops, and had been the premier system in the world from 2003 to 2005. After that, the systems from the USA retook the top position. In 2011, the "King" supercomputer designed by NEC became the top system again with a peak performance of 8.1624 Pflops.

China's Tianhe-1A system, designed and built by the National University of Defense Technology (NUDT), ranked first on the TOP500 list in November 2010, with a peak performance of 4.7 Pflops. In June 2013, the Tianhe-2 system, designed by NUDT as the successor of Tianhe-1A, achieved a peak performance of 33.86 Pflops, and has been the fastest system in the world until today. In the TOP500 list of June 2014, there were 76 supercomputers in China, and was only superseded by the USA with 233 systems. In recent years, there has been a strong competition between different countries to develop the fastest supercomputers in the world. The TOP500 list is also changing rapidly.

Note that the TOP500 list takes a simple classification approach to dividing supercomputer systems into star-shape, vector machine, MPP (massively parallel processing), and cluster.

## 1.2 Development and Architecture Classification of Supercomputers

## 1.2.1 Development of Supercomputers

The term "supercomputer" was first used in the early 1970s. The first generation supercomputer had the architecture of a SIMD array of processors. In 1972, the USAs successfully developed the ILLIAC-IV processor array, consisting of an $8 \times 8$ array of 64 processors. The floating-point performance of this system is around 100 Mflops (mega-flops, $10^6$ floating-point operations per second).

The second generation supercomputer was in the form of a pipelined vector machine. The Cray-1 system, which was designed by the Cray Company in 1976, is a typical example. The system includes 12 pipelined arithmetic units, each of which provides a different arithmetic function. The units can be divided into 4 groups, and perform the computation in parallel. The peak performance is around 160 Mflops. Cray-1 was the first commercial supercomputer system on the market.

Depending on whether the operands and results are stored in registers or memories, vector machines can be divided into two types: the memory-to-memory type vector machine, and the register-to-register vector machine. Vector machines in the early days were mostly memory-to-memory type vector machines, such as ASC designed by the TI company in 1972, STAR-100 designed by CDC in 1973, and CYBER-205 as well as ETA-10 designed by CDC in 1980 and 1986 respectively. In 1976, the CRAY-1 vector machine adopted the register-to-register structure for the first time. Due to its excellent performance when processing short vectors and the simpli fi ed instruction system, the register-to-register architecture became the mainstream of vector machines.

Third generation supercomputers have shared-memory multi-processor systems that parallelize the computation in the MIMD (multiple instruction multiple data) way. The Cray-1 system in 1976 was a single-processor vector machine. Since then, to improve the performance of vector machines, people continued to increase the number of vector processors in the system. The Cray X-MP/2 in 1982 had 2 vector processors, and the Cray Y-MP system in 1984 had 4 vector processors. The number increased to 8 for the Cray Y-MP 816 system in 1988, and 16 for the C90 system afterwards. Coming into 1990s, the number of vector processors within one supercomputer system increased to a few hundreds. In the twenty- first century, the number of vector processors increased to the scale of a few thousands.

In order to balance the performance of vector processing and scalar processing, the current parallel vector processor systems mostly use high-performance superscalar processors. As an example, the "Earth Simulator" designed by NEC company in the beginning of the twenty-first century consists of 640 nodes, each of which contains 8 vector processors, 16 GB memory shared among the processors, a remote access control unit, and a I/O processor. The peak performance of the Earth Simulator's vector processor system was 40.96 Tflops, staying in the top position of the TOP500 list for around 3 years (2002 -2005).

Fourth-generation supercomputers are MPP systems. The MPP system generally consists of tens of thousands of processors, and achieves a high performance through a high level of parallelism. We can build an MPP system using small nodes, NUMA, super nodes, or hybrid nodes with both vector and superscalar processors. In today's TOP500 list, MPP systems still take up a considerable portion.

Fifth-generation supercomputers are clusters, which are the most popular systems nowadays. The early clusters are homogeneous, while the current clusters are mostly heterogeneous, in the form of CPU + GPU or CPU + MIC (the Many Integrated Core architecture of Intel). The current number one system in the world, Tianhe-2 in China, provided a peak performance of 33.86 Pflops in the heterogeneous form of CPU + MIC.

## 1.2.2 Architecture Classification of Supercomputers

As supercomputers are highly parallel computer systems, we can apply the same classification of parallel computer architectures to supercomputers.

Depending on whether the parallel computing is achieved by SIMD (single instruction multiple data) or MIMD, and whether it uses shared or distributed memory, we can divide supercomputers into the following 4 types:

- shared-memory SIMD (SM-SIMD)
- shared-memory MIMD (SM-MIMD)
- distributed-memory SIMD (DM-SIMD)
- distributed-memory MIMD (DM-MIMD).

The early supercomputer systems perform parallel computing in the way of SIMD. As the processors are usually organized as an array, these systems are also called array processor. The memory in the array processor can be either shared-memory (SM-SIMD) or distributed memory (DM-SIMD). Array processor systems are usually special-purpose systems, and can only process one type of problems, but with high efficiency.

The single-processor vector machine has only one vector processor. However, the memory is shared among the vector processor, the scalar floating-point unit, and the scalar integer unit. Therefore, the single-processor vector machine is of the type SM-SIMD. These systems are general purpose, and provide a high efficiency when processing vector problems.

Currently supercomputer systems are mainly of the MIMD type. Multiple vector processor (MVP) systems contain MVPs that share the memory. Therefore, the MVP systems are of the SM-MIMD type. The symmetric multiprocessor (SMP) system is also of this type. Both MVP and SMP are considered as uniform memory access (UMA) systems, since the different processors in the system have the same access time to the memory.

The opposite of the UMA system is the NUMA (non-uniform memory access) system. The memory in a NUMA system is distributed, thus the memory access times for different processors depend on the location of the processor, and cannot be the same. Apparently, NUMA systems should be regarded as the DM-MIMD type. Note that the processors in a NUMA system could access remote memory by load-store instructions, so the system should have a uni fi ed logic memory space. Depending on whether the system provides hardware support for cache coherency, the NUMA systems can be further divided into CC-NUMA (cache-coherent) and NCC-NUMA (non-cache-coherent). If the memory part consists of only cache, we would have a COMA (cache only memory architecture) system.

If the processors in the system need to access remote memory by message-passing, then the system is called a NORMA (no remote memory access) system, which is also a DM-MIMD system. Different from NUMA systems, NORMA systems have multiple memory spaces, and each processor in the system is generally an independent computer node. The NORMA systems can be further divided into tightly coupled and loosely coupled systems. Clusters are generally loosely coupled NORMA systems, while MPP systems are usually tightly coupled NORMA systems.

MPP systems employ a large number of commodity computer nodes, and connect the different nodes using customized high-bandwidth and low-latency network. The memory is physically distributed, and need to use message passing to achieve inter-processor communication. MPP systems are tightly coupled parallel systems, and generally provide a good scalability. Typical examples of MPP include the Cray T3E and the IBM Blue Gene system.

In the cluster system, each node is an independent computer. The different nodes are connected using a commodity network. Each node runs a complete operating system, with an additional middleware to map different nodes into a uni fi ed pool of computing resources.

Figure 1.2 shows the structural classification of the supercomputer systems.

## 1.2.3 The Development Trend of Supercomputers

In the twenty- first century, with the increasing demand from various applications (weather/climate modeling, ocean modeling, nuclear simulation, vision, cognition, etc.), supercomputer systems are required to provide performance in the scale of Pflops or even Eflops (eta-flops, $10^{18}$ floating-point operations per second). Nowadays, we have quite a few Pflops supercomputers, mostly in the form of CPU + GPU or CPU + MIC heterogeneous clusters. The development of an Eflops supercomputer system is still in progress, with intensive competition between different countries.

Building an Eflops system is quite a challenge. The major reason is that, to achieve a performance in the scale of Eflops, we need to use a significantly larger number of processors and accelerators, which will lead to a tremendous increase in power consumption. Therefore, the development of processors with low-power and high-performance has become a key in the process of making the breakthrough. In addition to that, high-bandwidth and low-latency network, and heterogeneous architectures are also essential techniques to for building an Eflops-scale supercomputer.

We should also note that the development of E-scale supercomputers has to be supported by the corresponding developments in compilers and parallel programming languages. Programming tools are extremely important for improving the efficiency of the system and enabling the users to make use of E-scale computing resources.

![Image](/assets/img/posts/2026-03-05-The_Student_Supercomputer_Challenge_Guide_Chapter_1_artifacts\image_000002_78e6122123a523c026299c433b915dc9650d36a200b63bd50d9229a4f7cb19b5.png)

Fig. 1.2 Structural classification of the supercomputer system

## 1.2.4 The Rise and Development of Heterogeneous Architectures

Heterogeneous architectures date back to the 1980s. In many cases, the heterogeneous architecture enables the best match between the parallelism in the algorithm and the parallel computing power of the machine.

A heterogeneous computer system usually consists of three components: (1) a group of heterogeneous computer nodes, such as vector machines, MIMD machines, clusters, graphics processors, etc.; (2) a high-speed network (either a customized or commodity network) that connects these processors; (3) corresponding supporting software for programming the heterogeneous computers.

The basic idea of heterogeneous computing is to, firstly analyze the parallelism in the algorithm, secondly decompose the algorithm into different sub-tasks with different parallelism features, and thirdly map the different sub-tasks to different computing resources within the heterogeneous system, so as to achieve the best match between the algorithm and architecture and to minimize solution time.

Figure 1.3(i) shows an example of a loosely coupled heterogeneous computing system with distributed memory. The system consists of a vector machine, an MIMD machine, an SIMD machine and a workstation. Assume that within the algorithm, the portions of vector, MIMD, SIMD, and SISD computations stand for 30, 36, 24, and 10% of the total computation respectively. Assume that when running the right type of sub-tasks, the vector machine, the MIMD machine, the SIMD machine, and the workstation would achieve a speedup of 30, 36, 24, and 10 times respectively. T s stands for the execution time on a serial machine. Then the time for running the same job on the heterogeneous system would be $T_p = (0.3 T_s / 30) + (0.36 T_s/36) + (0.24 T_s/24) + (0.1 T_s/10) + T_c$, where $T_c$ is the time for communication. Assume that $T_c$ equals to 0.02 $T_s$ here, then $T_p$ equals to 0.06 $T_s$, which means we could get 16.67 times speedup over the original scenario.

![Image](/assets/img/posts/2026-03-05-The_Student_Supercomputer_Challenge_Guide_Chapter_1_artifacts\image_000003_497028672721d18ee0b8f4a9723d706774f9d4ef9a7993df8bcd2eea69af2177.png)

Fig. 1.3 Examples of tightly coupled and loosely coupled heterogeneous systems

Heterogeneous computing is widely used nowadays, in almost all the different scientific computing domains. Typical examples include image analysis, particle tracing, beam forming, and climate modeling. As all these different applications include different types of computation patterns, a heterogeneous system with different types of computing resources would be more suitable. We could take the image analysis system developed by the Hughes Institute and MIT as an example. The system contains three different layers, which corresponds to the requirements of the image analysis. The bottom layer is a SIMD bit stream network (4096-bit), which is used for pixel processing; the middle layer consists of 64 DSP chips, which perform pattern classification and other similar operations in the MIMD approach; the top layer is a general-purpose MIMD (coarse-grained) machine, which performs scenario, movement analysis, and other intelligent analysis functions.

The introduction of heterogeneous architectures into the supercomputer and the use of dedicated co-processors started in the twenty-first century. The major reason is that, since 1996, with the fast growing demands from applications, and the rapid development of VLSI and network technologies, the evolution of computer architectures was also accelerated. Although the superscalar techniques for exploiting instruction-level parallelism, dynamic prediction execution, and EPIC methods were successfully integrated into the commodity products, the design of the superscalar processors is becoming more and more complex, making it harder to further exploit instruction-level parallelism.

On the other hand, to provide a higher performance, the clock frequency of the processor chips becomes higher and higher, leading to the rise of power consumption and reduction of package density. Apparently, increasing the computing power by improving the processor's clock frequency and making use of instruction-level parallelism is not as efficient as before. Therefore, multi-threaded parallel computing and multi-core architectures become the new solutions. According to the practical results, multi-threading and data-level parallelism are becoming the key points for improving the parallel performance of the system. This change also demonstrates that parallel computing has become the mainstream technology of the current computer architectures.

As general-purpose processors are becoming more and more complicated and consuming more and more power, people start to think that it might be inefficient to continue pouring most resources on designing general-purpose processors. In contrast, special-purpose processors could be more powerful and energy-saving. Therefore, it could be quite helpful to offload more computing tasks to special-purpose coprocessors. The use of coprocessors is not a new technique. Since early days, microprocessors have been offloading a part of the job to floating-point coprocessors. The current term "coprocessor" indicates that these two kinds of processors are not symmetric. The coprocessors perform the computation for the master processors.

Using GPU for general-purpose computing is already a widely accepted method (this kind of GPU is usually called GPGPU). This is mainly because GPU has a high performance-price ratio. For example, compared with contemporary high-performance CPUs, the bandwidth and compute power of GPU could be one order of magnitude higher. In addition to that, the upgrade of GPU devices is also faster than CPU. The computing power is usually doubled every 6 months. As the programming tool of GPU is also improving quickly, GPU has a bright prospect as a general-purpose computing processor.

There are several reasons for the amazing performance of GPU. From the economic perspective, the performance demand of GPU is driven by the video games industry. From the technical perspective, GPU is designed for graphic processing, which is a kind of intensive computing application with a lot of data parallelism. Therefore, it would be much easier for GPU to improve floating-point performance by adding more processors. Moreover, as a coprocessor, GPU could largely ignore a large part of functions that the CPU needs to handle, and does not need to spend resources on branch prediction, and complex instruction scheduling units. The early GPU devices were only designed for supporting graphic processing, with a fi xed hardware pipeline and support for single-precision floating-point operations only. In recent years, with new demands from game developers, the GPU hardware supports more and more different ways of programming. The result is that the GPU can now support general-purpose computing. The current GPGPU devices include units for double-precision floating-point and integer operations.

A modern GPU is like a massively parallel fi ne-grained multi-core chip with on-chip DRAM. For example, there are 8 streaming multiprocessors inside the NVIDIA GeForce 8800, and each streaming multiprocessor includes 16 SIMD cores.

Although the hardware architecture is of SIMD type, the programming model does not need to be exactly the same. The CUDA (Compute Uni fi ed Device Architecture) programming interface developed by NVIDIA is almost a subset of the C programming language. The remarkable advantage of GPU is that it could efficiently perform parallel floating-point computations. Figure 1.3(ii) shows the tightly coupled heterogeneous computing system with shared memory.

MIC (Many Integrated Core) is the latest parallel architecture developed by Intel Corporation, and is mainly applied in applications with a high level of parallelism in high-performance computing, workstation and data-center environments.

Compared with traditional processors, Intel MIC has smaller cores and hardware threads, and wider vector units, which could promote the entire performance more efficiently and meet the requirement for high parallelism. In addition, MIC provides a good compatibility, and supports programs of standard C, C++, and FORTRAN. Moreover, MIC and Intel Xeon CPUs share the same set of tools, compilers, and libraries. Since over 80% of supercomputers in the world use Intel Xeon CPUs, most of the developers can take advantage of their previous Xeon CPU experience.

In the TOP500 list of June 2014, there were 62 supercomputers that were heterogeneous systems with a certain kind of coprocessors. Among the 62 systems, 44 used NVIDIA GPU as a coprocessor, while 17 used Intel MIC (also known as Intel Xeon Phi) as the coprocessor.

## 1.3 Current Status and Challenges of Supercomputing Applications

The quality of supercomputing applications largely depends on the quality of parallel programming. However, parallel programming brings many more challenges than traditional serial programming. The major difficulty is that when doing parallel programming, programmers face a lot of different options that they do not need to consider in serial programming.

The programmers need to consider: which algorithm template to use; which parallel programming model to use; which parallel programming language to use; and what kind of supercomputer platform to use, etc. For current supercomputers, the parallelism is no longer hidden from the programmers. The software cannot achieve automatic performance boost along with the increase of CPU clock frequency. The programs would not have better performance if they were unable to use multiple cores within one processor.

## 1.3.1 Current Status of Supercomputing Applications

After more than 40 years of development, there are lots of improvements that have been achieved in parallel programming methods and techniques, detailed as follows.

## 1.3.1.1 Five Kinds of Parallel Programming Algorithm Templates Have Been Summarized

As mentioned earlier, parallel programming is a complex task. The programmer should firstly decide which kind of parallel algorithms should be used to solve the

problem. Thankfully, although the history of the parallel algorithms is shorter than that of serial algorithms, there are still some well-de fi ned templates for users.

## (1) The stage-based parallel algorithm

In this algorithm, the program is divided into a number of stages. Each stage consists of the computing part, and the communication part. For the computing part, each process performs the computation independently; for the communication part, the processes exchange data by message passing. After that, the program goes to the next stage. Figure 1.4(i) shows the workflow of this algorithm.

The main advantage of the stage-based parallel algorithm is that it is easy for us to analyze the parallel performance of the program, due to the separation of computing and communication. However, the disadvantage is that we are not overlapping the computing part and the communication part, and it is generally difficult for to achieve good load balance among different processes.

There are two special cases of the stage-based parallel algorithm: synchronous iteration and asynchronous iteration. In synchronous iteration, different stages become a sequence of iterations inside one loop. As barrier synchronization is used, the stage $( j + 1)$ could not be started unless all the processes have finished stage $j$ . However, in asynchronous iteration, since there is no barrier between different processes, the next stage for one process could be started as soon as this process finishes its current stage. We should notice that if there is any data dependency between processes, asynchronous iteration is not a suitable choice.

## (2) The divide-and-conquer algorithm

The core concept of the divide-and-conquer algorithm is to continue to divide the problem into several sub-problems in a recursive way, until the sub-problem could be further decomposed. After the divide process, we assign these tasks into different nodes of a supercomputer to compute. After the computation is finished, we merge the results in a reverse pattern to the way that we divided the problem. Figure 1.4(ii) shows this process.

## (3) The macro-pipeline parallel algorithm

This algorithm is based on function decomposition of the program. Firstly we divide the whole problem into several parts according to its feature, then we assign each part into one node. Communication in this algorithm is quite simple since data can only be passed between neighboring nodes in the pipeline. Figure 1.4(iii) shows the basic idea of this algorithm.

## (4) The master-slave parallel algorithm

In this algorithm, the master process is responsible for the basic serial parts of the program, and for allocating tasks of the program into other slave processes. When one slave process completes its own work, the main process would be noti fi ed, and another task will be allocated to that process if necessary. In addition, the master also takes charge of summing up the sub-results. While the design of the master-slave algorithm is quite straightforward, the main process might become the

![Image](/assets/img/posts/2026-03-05-The_Student_Supercomputer_Challenge_Guide_Chapter_1_artifacts\image_000004_5a94eca5e197404b39d20a792e3558010e7186586d1f7644acee74941d39f621.png)

Fig. 1.4 Five kinds of parallel programming algorithm templates

bottleneck of the entire program. Note that in this algorithm, we generally take the static load balancing method, with the task allocation speci fi ed at the very beginning. Another feature of the algorithm is that the communication only exists between the master process and the slave processes. Figure 1.4(iv) shows the general workflow of this algorithm.

## (5) The job-pool parallel algorithm

This algorithm is commonly used in the shared-variable model, and is generally implemented using global data structures. There might be just one job inside the pool at the beginning, and any of the idle processes that have been initiated can obtain a job from the pool and execute the job. During the execution of jobs, more jobs could be generated and put into the pool. When the pool becomes empty and processes are no longer generating new tasks, the execution of the program is finished. It is easy to achieve load balance in this algorithm, since the jobs are generated and allocated dynamically. The disadvantage is that it is difficult to achieve an efficient implementation for the processes to access the job pool concurrently. The job pool is usually implemented as an unordered set, a queue, or a priority queue. The algorithm is shown in Fig. 1.4(v).

## 1.3.1.2 Various Types of Parallel Programming Languages Are Available for Developers

There are various types of parallel programming languages available for developers which include:

- (1) Multi-threading programming languages for the shared memory model, such as POSIX Threads, Java Threads, OpenMP.
- (2) Message passing programming languages for the distributed memory model, such as MPI, PVM.
- (3) Parallel programming languages for data-parallel designs, such as HPF (High Performance Fortran).
- (4) Parallel programming languages for heterogeneous platforms, such as CUDA developed by NVIDIA.

## 1.3.1.3 Three Parallel Programming Models Are Available for Developers

There are three parallel programming models available for developers that include:

- (1) The shared-variable model.
- (2) The message-passing model.
- (3) The data-parallel model.

In conclusion, with the rapid development of supercomputers as well as parallel programming techniques, supercomputers are widely used nowadays. In some application fi elds, such as the Human Genome Project, supercomputers have made significant progress in both scientific research and industry innovations, which enhances developers' confidence to apply supercomputers in even more domains.

## 1.3.2 Challenges of Supercomputing Applications

In supercomputing applications, the following problems are the key challenges the programmers should pay special attention to:

## 1.3.2.1 How to Write Highly Scalable and Portable Parallel Programs

The performance of a program is not only related to the application, but is also related to the hardware. It is hard to keep a linear speedup of the program when we increase the number of processors to a certain level. However, the number of cores available to programmers continues to be doubled every 18 months. Therefore, a successful software should be able to scale its performance with the increasing number of cores, in a similar way to the previous scaling of serial programs' performance with the increase of CPU clock frequency. Besides scalability, portability is another important feature of software that would be used long term, so that we can apply the same software on different supercomputers with reasonable performance. When writing a parallel program, we should take scalability and portability as the most important metrics from the very beginning of the design process. Following this principle, the resulting software would have a much longer life cycle.

## 1.3.2.2 How to Enable Automated Use of Effective Parallel Programming Techniques When Writing Parallel Programs

The automated use of parallel programming techniques during the programming process can largely improve the efficiency of the program. The current major parallel programming techniques that are proven to be effective include: incremental development; use of parallel data structures; efficient use of cache and other fast buffers in the system; extending the decomposition scheme from 1D to 2D or even 3D, so as to improve the parallelism and reduce communication and synchronization; and exploring different allocation methods, such as static/dynamic allocation, block allocation.

## 1.3.2.3 How to Enforce the Use of Parallel Programming Design Principles During the Programming Process

Why should we focus on the design principles? It is because, while the hardware architecture and the programming tools of parallel systems keep changing, the design principles will never be outdated.

The key principles that we should follow include the scalability and portability principle, the abstract model of parallel computing, static/dynamic allocation principles, and the owner-compute principle.

It is a challenging task for programmers to follow these principles when doing parallel programming.

## 1.3.2.4 How to Employ Suitable Optimization Techniques

Major optimization techniques include: an efficient utilization of the different components in the memory hierarchy, especially the fast buffers, such as cache; assembly-level code optimization.

## 1.3.2.5 How to Promote Interdisciplinary Collaboration

Supercomputing applications are usually related to other disciplines, besides computer science. Therefore, only with the close co-operation between the different disciplines, can we achieve the most efficient application software.

Finally we should mention that, with the development of supercomputers and corresponding applications, a new discipline called "Computational Science" is gradually maturing. With the interdisciplinary collaboration with other scientific domains, computational science and its related supercomputing technologies have been developing fast in recent years. The efforts on these areas would be a strong driving force for the improvement of technology competitiveness and innovation. The research and development of the supercomputer architectures are a key component in this process.
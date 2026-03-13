---
title: "The Student Supercomputer Challenge Guide Chapter 2: Supercomputing System Construction and Power Management"
author: khw
date: 2026-03-13
categories: [Study, Supercomputing]
tags: [LAB]
pin: true
math: true
mermaid: true
---

# Chapter 2 Construction and Power Management of Supercomputing System

## 2.1 Components of a Supercomputing System

On the TOP500 list in November 2016, most HPC systems are either clusters (432 machines, 86.4% of the total) or MPP systems (68 machines, 13.6% of the total). The cluster architecture is now the most popular architecture among mainstream systems.

Usually, a typical HPC cluster system consists of five categories of computing (or network) devices and three types of network. The five categories of devices mainly refer to the login nodes, management nodes, compute nodes (including thin and fat nodes), switching equipment, I/O, and storage nodes. In addition, many current CPU/GPU/MIC heterogeneous HPC systems include the GPU/MIC accelerator nodes.

The login node serves as the gateway for users to access the cluster. Users typically log onto this node to compile and submit the job. The login node is the only entrance for external access to the powerful computing or storage capacity of the cluster, and the key point of the entire system. In order to guarantee the high availability of the login nodes, we should apply fault-tolerance approaches, such as hot backup dual-node technology, or at least RAID (Redundant Array of Independent Disks) technology to ensure the data security of the user nodes.

The login node generally does not require high computing power. In general, the entire cluster only needs to configure several rack-mounted servers as login nodes according to the requirements.

The management node is used to control all kinds of management strategies in the cluster. It is responsible for monitoring the robustness of each cluster node and the status of the interconnect network. Generally, the cluster management software also runs on this node.

As a result of the powerful processing capability of the modern servers, the job scheduler program (such as PBS) can also run on this node. Generally, the management node is also the key point of the computing network. If the management node fails, all the compute nodes would fail, and the submitted jobs would hang up. Therefore, the management node should also have hardware-redundant protections.

Similar to the login node, the management node does not require high computing power. In the entire cluster we generally only need to configure several rack-mounted servers (shown in Fig. 2.1) according to the requirements.

The compute node is the essential part of the cluster that performs the computations. The configuration of the compute node is generally decided by both the needs and budget. We can further classify the compute node into thin nodes and fat nodes.

Due to budget considerations, the main computing power of the cluster is provided by large numbers of thin nodes of the same configurations. Due to the large quantity of thin nodes, energy and space saving become some of the most important concerns of the customers. The blade server (shown in Fig. 2.2) has become the mainstream.

The fat node is usually used for the special applications that are difficult to perform domain decomposition or require particularly large memory space. The fat node (Fig. 2.3) generally uses more than 4 ways of processors, and large size of memory. Accordingly, the price of the fat node is also relatively higher.

The heterogeneous node, in recent years, it has already become a hot topic to adopt the heterogeneous computing system for general-purpose scientific and engineering computing. The current heterogeneous nodes usually adopt both CPU and GPU or MIC accelerators (Fig. 2.4). Making use of accelerators for parallel computing can significantly improve the computational efficiency in many cases.

The switching equipment, all the nodes in the cluster are connected through the network. Information and data are exchanged between the nodes via the switching equipment. In a large cluster, the switching equipment of the computing network often uses the large switches with hundreds of ports (Fig. 2.5).

The I/O node and storage device, to enable highly-efficient parallel processing, each compute node generally needs to access the data in parallel as well. Meanwhile, the concurrent computing processes generate large volumes of important data, and require professional storage devices that provide a large storage

Fig. 2.1 The rack-mounted server

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000001_d9dcebfd85d8b8ea0b3c5a10739435efc890ccf30e63a5796d1854e5319570c9.png)

Fig. 2.2 The blade server Inspur NX5440

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000002_54e464d77eb19185bf62637b3e4ad8d820565230bdd23cb7a5e5830c75e09284.png)

Fig. 2.3 The 8-way fat server node TS850 from Inspur

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000003_efd3c34cfeb7ceb0d043bdf5c98f77cfc825ef9cec316e0d891a01a413af86c8.png)

space and security functions. The I/O node provides the interface between the storage devices and the compute nodes, to ensure the synchronization of data access.

The storage system does not only serve the role of data backup, but also provides the function of improving the read/write bandwidth during the computation process. In most software, we take the approach to keep the most frequently used data in memory, and to keep the less frequently used data on the disk storage. Even though the data stored on the hard disk is used less frequently than that stored in the memory, the amount of hard disk data is huge and needs to be updated at run time during the computation process. Therefore, we require a high read/write bandwidth.

Fig. 2.4 Inspur Yitian series heterogeneous supercomputing server NF5588

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000004_2ab804b889d01c247c3c1be8d682add5fd83958470f630af2acf439bb8d2fb18.png)

Fig. 2.5 The large all-gigabit-Ethernet switch

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000005_f3bff44ac39fbda3a11ce3d4134a613a4a90283ef6cfe772afc7c1cb3e4c3bea.png)

Generally, the local hard disk access speed cannot meet the requirements, and a specialized file storage system, as shown in Fig. 2.6, is needed.

The management network is used for the interconnection of the management nodes, compute nodes, and I/O nodes. Since the nodes connected by the management network are within the cluster, we do not require high bandwidth and low

Fig. 2.6 Inspur AS10000 mass storage system

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000006_9efd3c716995c46220c8d8d25a66f196b5a47a58812b6184f302d0f2ac41d92b.png)

latency, and we can tolerate a certain degree of over-subscription. Based on the above considerations, the gigabit Ethernet is a suitable candidate.

The computing network is used for the interconnection between the compute nodes and is specifically used for the inter-process communication (IPC) during the parallel computing process. One key component of parallel computing is the capability to exchange information with the other nodes in the cluster, which is often called IPCs. It requires a high-performance network for rapid exchange, thus requiring low latency and high bandwidth. The interconnect structure and bandwidth of the system are important factors that determine the system architecture, performance, and applicability to different applications. For example, mesh and multi-dimensional mesh structures are suitable for computational fluid dynamics (CFD) and other similar scientific computing applications. In these applications, the patterns in data partition and communication provide a good fit for the mesh structure, and can generally achieve a good efficiency of the system.

For large-scale computing tasks, parallel computing is the only solution to dramatically increase computing speed. With the development of CPU and memory technologies, the computing capability of a single node becomes more and more powerful. Therefore, in order to maximize the performance of each single node, we require high bandwidth and low latency of interconnect. The higher the bandwidth and the lower the latency the higher the computational efficiency that can be achieved from the entire system.

The computing network usually adopts the gigabit Ethernet, the IB networks or the 10-gigabit Ethernet.

The storage network provides the data access service for the nodes in the HPC cluster. In the HPC field, according to the different storage and access mode, we have a number of different storage models. At the lowest level, there are two ways to access the data: one is the file-level data access provided by the external file system, including NAS; the other is the data-block-level access, including DAS or SAN. SAN can either use IB storage or the Fibre Channel based on SCSI or SCSI RDMA (SRP) protocol.

## 2.2 Power Monitoring and Management of a Supercomputing System

With technology innovations, the server's initial cost continues to drop, while the power and other operation costs continues to rise. Therefore, improving the power efficiency of a computer system is a new focus for the development of supercomputers.

While the computing power of a supercomputer is significantly higher than normal computing systems, the energy consumption is even more astonishing. The power consumption of Tianhe-1 is 6000 kW, which costs up to 50 million CNY (Chinese Yuan) for the electricity every year. For a 50 peta-Flop supercomputer, the power consumption is estimated to be 20,000 kW, which is equivalent to 0.1% of the total electricity consumption of Shanghai. Facing the growing pressure on increasing energy consumption and the demand to be environmentally friendly, the design of supercomputers is increasingly focusing on the energy efficiency, that is, the computing capability per watt. To improve energy efficiency, we need to focus on two aspects: one is to design more energy-efficient supercomputer components; the other is to design a more energy-efficient cooling method.

Energy optimization is currently gaining extensive attention from the industry. Some people focus on the new system architectures, while most of us are still focusing on the refinement of the existing architectures.

Among the main components of the server, the processors consume a large portion of the entire power cost. Processor power consumption management methods include Dynamic Voltage Frequency Scaling (DVFS) and the dynamic

sleep mechanism. With the maturing of low-power technology, almost all of the current mainstream operating systems have built-in power management strategies.

In most scientific computing applications, the power consumption of computation, communication, and synchronization are the major parts of the power consumption, with the I/O accounting for a small portion. Therefore, the processor, the power supply modules, the memory, and the fan become the major consumers of power. Figure 2.7 shows the power consumption ratio of different components when the server is in idle mode. Power consumption can be different for different servers and different workloads. However, in most cases, the power consumption of processors still accounts for the largest portion.

Currently there are a lot of power-reduction techniques, and a variety of effective power-management schemes have been proposed based on these techniques. These management schemes could be divided into two categories according to their mechanisms.

- (a) Dynamic Resource Sleeping (DRS). The key idea of this technique is to shut down idle resources, such as components, devices or nodes to save energy, and awake them when needed. Currently most mainstream processors provide support for the DRS technology. The detailed standards of Resource Sleeping are described in the Advanced Configuration and Power Interface (ACPI). In addition to that, some memory devices also support dynamic shutdown. The PCI power management standards also describe the general ways to dynamically shutdown PIC devices. In a supercomputer system we could also put some idle nodes to the sleeping mode.
- (b) Dynamic Speed Scaling. The nature of this technique is to dynamically adjust the executing rate of devices. There is a large amount of communication and synchronization in parallel computing. Therefore, in many cases, the fast components would have to wait for the slow components. In such scenarios, it

Fig. 2.7 Power consumption of different parts of a server in idle mode

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000007_a958ff3262156274c42630174668b41d965882ff1a53925a0ad7856551d815ab.png)

is not necessary to keep the high rate of the fast components. By reducing the rate of fast components, we can reduce the power consumption while providing the same performance. The DVFS of processors is a typical DSS approach. Some memory and disk devices also support dynamic adjustment of the running rate.

Almost all current commercial processors support DSS and DRS as mentioned above.

In real supercomputing systems, processors will not always be busy processing computation tasks. It is would be a waste of energy to keep it running when there is no computation happening.

Therefore, the DRS techniques are introduced. Different processors support different sleeping modes with different power consumptions. In general, it takes more time and energy to wake up from a sleeping mode that consumes less energy. For instance, the Intel Xeon CPUs support the Enhanced Halt State technology. Besides the normal running state C0, the processor could also be in the sleeping mode C1, or even the enhanced deep sleeping mode technique. According to the idling record of the processor, the system would decide whether to put the processor into sleeping mode and will choose the right sleeping mode accordingly.

According to the experimental results, for test cases, using the C1E sleeping mode would take less energy and longer time tofinish the job than using the C1 sleeping mode. However, for compute-intensive applications, the longer waking-up time of the C1E mode affects the efficiency more seriously. In such cases, while the C1E mode takes more time to compute, the energy consumption would also be more.

The optimization results by adjustment of the frequencies of the processors largely depend on the characteristics of the application. For communicationintensive applications, underclocking could bring about a 10% reduction in power consumption. However, for compute-intensive applications, underclocking might even lead to increased power consumption.

Therefore, we should make adjustments of the processor frequencies according to the computation and communication patterns of the parallel program. In supercomputing, there is still a big potential in power optimization by dynamic speed scaling of processors.

There are two levels of power consumption monitoring for clusters. Thefirst is at the cabinet level. A PDU is installed in each cabinet to measure the power consumption within the cabinet. The monitoring software can then collect the data from all the cabinets to achieve a picture of the entire system. This also provides the direct data for estimating the power consumption cost of the system.

The second level is at the node level. We could either employ professional monitoring software or some simple tools in the OS to perform the monitoring of the node.

## 2.3 Building a Performance-Balanced HPC System

### 2.3.1 A Performance-Balanced HPC System

According to Amdahl's law, when the CPU performance increases by 10 times with the I/O performance remaining the same, the overall performance of the system can only increase fivefold; similarly when the CPU performance increases by 100 times without improving the I/O performance, the overall performance of the system can only increase tenfold. Therefore, the balance among the various components of the system is extremely important. If the poor performance of some components becomes a bottleneck of the system, the overall performance will reduce, and we will not be able to take full advantage of the other components in the system.

Generally speaking, there are two aspects about design balance of a supercomputer system. One is the architectural balance among the capabilities of different hardware components in the system. The other is the software level balance, which mainly refers to the load balance of different nodes achieved through effective management of different resources.

The supercomputer mainly consists of the computation components, the storage components, and the interconnect components. To achieve a balanced system requires coordination of the performance of these three components, so as to avoid both redundancy and bottleneck under the specific workload.

To build a well-balanced HPC system, we need to take the following aspects into consideration.

#### 2.3.1.1 The Intra-node Configuration

- (1) The homogeneous CPU type. We should keep the intra-node configuration as consistent as possible in order to improve the data exchange capabilities and the efficiency of resource utilization. For example, if the node is 2-way, we should keep the same memory configuration for both processors, so as to avoid the memory difference, the performance difference, and resulting reduction of the processing capability of the node. Specifically, for a 2-way node, deploying 24G memory for both processors will bring better efficiency than deploying 24G memory for 1 processor and a 12G memory for the other.
- (2) The heterogeneous type. In a heterogeneous computing system using both the CPU processors and GPU accelerators, the host memory should be in general larger than or at least equal to the GPU onboard memory. One good idea is to build a unified and shared memory space between the CPU and the GPU so as to achieve seamless interaction between the two. We should keep a balanced ratio between the number of CPU cores and the number of GPU cards, as well as a good ratio between the GPU onboard memory bandwidth and the host memory bandwidth.

#### 2.3.1.2 The Inter-node Configuration

For the compute nodes of the same category, we should keep a same balanced configuration for major hardware parameters, such as the number of processors, the number of cores, memory size, and the architecture of the node (homogeneous or heterogeneous).

#### 2.3.1.3 The Network

Similarly, connections between the same types of nodes should adopt the same kind of interconnect, such as gigabit Ethernet, 10-gigabit Ethernet, IB networks.

For switches, we should either use full-bandwidth cascading or half-bandwidth cascading for the entire system. Otherwise, the different bandwidth in different connections might lead to communication congestion.

#### 2.3.1.4 The Configuration of Nodes Executing the Same Task

For nodes that execute the same task, we should keep their configuration consistent to achieve a faster speed when processing and exchanging the data. If the hardware consistency cannot be achieved due to practical reasons, we can use cluster management software to allocate the job to the compute nodes with the same configuration, so as to avoid performance bottleneck caused by certain nodes within the cluster.

#### 2.3.1.5 The Balance Among Different Devices

We should also consider the balance between the compute nodes, the I/O nodes, and the storage devices. The storage system within a HPC system is responsible for not only the data backup but also the increase of the read/write bandwidth during the computing process.

For applications that mainly perform computations, and seldom store or read data from disk, it is unnecessary to deploy a high-end I/O node. In contrast, for applications that involve large-volume data read and write, such as the genetic and geophysics exploration applications, we need to deploy a powerful I/O node to satisfy the high I/O demand.

#### 2.3.1.6 The Power Consumption

Based on the practical situation of the computer room, when designing the system layout, we need to distribute the devices in a balanced way, so as to avoid the power consumption hot-spots in the room.

Under the constraint of satisfying all function requirements of the system, we should combine devices with high-energy consumption and devices with low-energy consumption, and combine the high-density devices with the low-density devices. For example, putting the blade servers and rack servers into the same cabinet can avoid huge power consumption differences among different locations and the possible local overheat or local waste of power resource.

In addition, to make efficient utilization of resources, we could place the high-density equipment with high-power consumptions close to the cooling equipment in the computer room.

### 2.3.2 How to Manage the HPC System

HPC systems have been widely used in many different domains. Meanwhile, its management has also become an important issue. With the continuous improvement of the computing performance, the scale of the compute node has also kept on expanding. A typical HPC system nowadays can have thousands of compute nodes, different kinds of computing resources (CPU/GPU/MIC), a complicated configuration of management software, hundreds of users and jobs running in parallel. All these quantitative increases would eventually lead to the qualitative change on the challenge of management for HPC systems.

In order to achieve higher efficiency, an HPC system requires an integrated management of the computing resources, network resources, storage resources, energy resources, and job executions, as shown in Fig. 2.8. For the computing resources, the heterogeneous accelerators and the multi-core architectures of the processors have increased the complexity of the management. For the network resources, due to the increasing scale of the interconnection, the performance impact of the mapping from the interconnect topology to the parallel program

Fig. 2.8 The overall structure of an HPC system

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000008_75b1759865f4aa74bbe2332303b098535d477e70c6263353b851167f620bb3d9.png)

becomes an important factor. For the storage resources, the increase of the different levels within the system, the increase of levels of cache within the multi-core processor, and the distributed structure of the file system all bring tough challenges for storage management. For job scheduling, the increased types of parallel programming models require more effective management of the jobs of multiple models.

In order to achieve high availability of the HPC systems, the management needs to include flexible extensions, which can support a quick adjustment according to the change on system management objectives and application objectives. Meanwhile, the growing demand from the parallel applications brings a heavier and more complex workload to the large-scale processing system. Moreover, applications from different domains generally demonstrate different parallel features in both time and space. All these different challenges result in a significantly increased complexity of job scheduling, making it difficult to scale the throughput with the scale of the system. Therefore, we need to develop a scheduling approach that is more suitable for the large-scale parallel processing system features and the application characteristics, and to enhance the adaptability of the scheduling policies, so as to achieve a high efficiency.

As for China's local HPC manufacturers, they need to focus more on the innovations in the aspects of the applications and product technologies, and apply the HPC systems into the practical industries, such as medicine, chemistry, materials, physics, CAE, oil and gas, weather forecasting, environment protection, academics, and scientific research. In addition, HPC manufacturers also need to develop a fair amount of management software (e.g. cluster monitoring software, cluster job scheduling software, system deployment software, and system backup software) so as to make the operation more convenient for the users, and to improve the practical efficiency of the system.

### 2.3.3 Monitoring Management

The cluster monitoring software mainly achieves a 1-to-1 mapping of the entire system, which provides a unified platform for the system administrator to monitor each single node in the cluster. Through the software, the administrator can choose to monitor the nodes as groups, or to monitor 1 specific node. The current mainstream cluster monitoring software generally adopts the B/S structure to achieve highly efficient remote management, and provide functions such as the group management, parallel operation, 1-to-1 mapping, and quasi-3D graphical interface. Some monitoring software even provide functions such as accounting, and raising alarms, thus leading to a significant decrease in users' management costs and improvement of management efficiency.

Through the monitoring of various components and patterns of the cluster system, such as the status of processor, memory, network card, workload, memory

page, process context, swap partition, disk capacity, and disk I/O, the monitoring software can collect real-time data to help the cluster administrators obtain the basic data, and to analyze and manage the system.

The cluster monitoring software provides an integrated interface for the administrator to perform centralized management of the cluster system and the related resources. For example, the administrator can create a user or user group, and turn off or restart a cluster node through the software interface, rather than operating on the physical machine. The monitoring log provided by the software also helps the administrator to locate and resolve problems in a better way.

A typical HPC cluster can be very complex in structure, and generally consists of a large number of different nodes and devices. Therefore, as shown in Fig. 2.9, it is impractical to rely on the administrators to manually monitor and fix the problems of the system. Instead, we can use an alarm system, which can actively scan the cluster system for failures and automatically report the problems to the administrators through light, sound, email, or text messages. In this way, the administrator of the system can quickly identify the problematic node and perform the maintenance work. The alarm system is a smart combination of the personnel, the technology, and the equipment, and is regarded as the most effective measures so far. Users can get high reliability of the system with a relative low cost.

Fig. 2.9 Inspur ClusterEngine cluster monitoring interface

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000009_258491be0d72f183c3b065d291645582d73341d401f57e24f98b5f386890b318.png)

### 2.3.4 Job Scheduling

The cluster platform is widely applied in a variety of application domains, including the basic science research (such as computational chemistry, geophysics, artificial intelligence), public service (such as numerical weather forecasting, earthquake prediction), industrial and engineering computing (such as aerospace, automobile design, geological exploration, etc.), and data processing (such as finance, e-government). Therefore, the job scheduling system must be able to support various applications with different characteristics, ranging from engineering computing applications to basic scientific research computations. Based on the above considerations, the job scheduling system needs to provide proper interface and background support for both commercial software and open-source software, so as to ensure the stability of system when running different applications.

The job scheduling system is the channel for the users to access the HPC platform. We use the login node as the only interface to access the computing service, and also a barrier to isolate the users from the back-end system to improve reliability and manageability. At the management network layer, the job scheduling system provides a communication platform for the deployment, monitoring, scheduling, and management of the system. At the computing network layer, the job scheduling system provides the data communication between the high-performance application and the parallel-computing applications, so as to reduce the latency and to increase the bandwidth. At the storage network layer, the job scheduling system provides the high I/O throughput and high-bandwidth communication access for the storage servers and storage devices to ensure the high concurrency and large throughput of the storage usage. The storage access from other nodes does not merely depend on the storage network but also depends on the management network or computing network between the I/O nodes and other nodes (depending on whether the I/O nodes use the management network or the computing network to provide the I/O service).

The traditional job scheduling systems use commands to schedule jobs. The scheduling process can involve numerous and complicated combinations of commands therefore users need to be computer experts with advanced skills. The current job scheduling systems adopt a unified graphical interface to submit jobs, and the operation is more convenient and less error prone. The job scheduling systems provide concrete support for the running and scheduling of large-scale and complicated computation jobs, improve the efficiency of the cluster, and reduce the operation cost significantly.

Nowadays, the mainstream job scheduling software usually uses the B/S architecture, and conducts the operations through the browser (IE, Maxton, Firefox, etc.). Through the scheduling software, we can manage the hardware and software resources of the system and the jobs submitted by users. According to the cluster resource usage, we can then schedule the jobs to improve the resource utilization rate and the computation efficiency.

### 2.3.5 The 10-Tera flops HPC System Design

A computation capability of 10 tera flops (Tflops) is generally considered as the starting point for a modern HPC system. A cluster with a performance of over 10 Tflops already demonstrates the basic features of an HPC system. On the other hand, as 10 Tflops is only the starting point, the structure of the system does not need to be as complicated as a 100-Tflops system or a system with even higher performance. For example, the management node and login node do not need to be physically separate. Instead, they can share the same physical node with the compute node. In addition, the requirements on network and uninterruptible power supply are also relaxed. In general, we can build a 10-Tflops system in two ways: the CPU homogeneous approach and the heterogeneous approach.

#### 2.3.5.1 Build the 10-Tflops HPC System

When building a 10-Tflops HPC system, we should mainly consider the following aspects:

##### Computing Power

As the main part of the entire cluster, the compute nodes are the basis of the entire system. The performance of the compute nodes directly determines the overall performance. Therefore, when choosing the processors, we should select the processors with more computing cores within the same chip size, and the processors with higher performance and less power consumption, such as the 22-nm Intel Ivy-bridge processors.

The compute node is a complete computer system that involves the interaction among all various components. In high-performance computing, almost all the computing software relies heavily on the frequent interaction between the CPU and the memory. The high performance of the CPU can only be made full use of when matched with the suitable memory. Therefore, when building the system, we should select the large-capacity and high-bandwidth memory to improve the performance and provide the fast memory access channel for each CPU.

##### Management

In terms of management, the HPC system handles different hardware devices, complex software configurations, and an increasing number of users and jobs. A good cluster monitoring and management system for the system can help to improve the management efficiency. The management system should include functions for the management of each node, the management of operating system and software, and the management of the cooling system and the room environment, for example.

##### The Network

The current mainstream of the HPC network includes the IB (InfiniBand) network and the GbE (Gigabit Ethernet) network.

The InfiniBand is a unified interconnection structure, and can be used for the storage I/O, the network I/O, and the IPC. Due to its high reliability, availability, scalability and the high performance, the InfiniBand can be used to interconnect the disk array, SANs, LANs, servers and the cluster servers, and can also be used to connect to the external network (such as WAN, VPN, Internet). Since the InfiniBand can provide the high-bandwidth and low-latency transmission within a relatively short distance and support the redundant I/O channels in a single or multiple interconnected networks, the IB network can still keep the data center running when having partial failures. In recent years, the IB network has become more and more popular among the HPC systems. In the June 2013 TOP500 ranking list, the number of machines using the IB network reached 205, accounting for 41% of the total.

The GbE network is defined by the IEEE 802.3-2005 standard. The GbE network used to be the most popular interconnect in the HPC systems. However, with the rapid development of the IB network, the market share of GbE has decreased slightly. In the 2013 TOP500 list, 140 machines adopt the GbE network, accounting for 28% of the total.

##### The Storage

The HPC storage system not only plays the role of data backup but can also improve the read/write bandwidth during the computation process. The current mainstream storage systems include DAS (Direct-Attached Storage), NAS (Network Attached Storage), SAN (Storage Area Networks), and parallel file system storage.

In a DAS storage mode the RAID disk array is mounted directly onto the server network systems, the storage devices are directly connected to the server through the cable (usually the SCSI interface cable), and the I/O request is sent directly to the storage device. Based on the server, the DAS storage is a hardware stack without any storage operating system, thus needs the support from the corresponding server operating system.

In the NAS structure the storage system is no longer attached to a particular server or client by the I/O bus, but is directly connected to the network through the network interface and accessed by the network. The NAS storage will store the files from different platforms onto 1 NAS device, enabling the network administrators to centrally manage large amounts of data and reducing the cost of maintenance.

SAN is a high-speed network or sub-network providing data transmission between the computer and the storage system. A SAN network consists of the communication structure responsible for network connections, the management layer responsible for organizing the connections, the storage components and the computer system, thus ensuring the reliability and effects of the data transmissions.

The parallel file storage system is mainly used to provide the high-bandwidth storage capacity management. The parallel file system is a network file system used in the multi-node environment. The data of a single file is distributed onto the different I/O nodes in a striped form and supports concurrent access from multiple processes on multiple machines. The parallel file system also supports the separated storage of the metadata and the raw data, and provides a single unified directory space.

##### The Operating System

As for the operating system, from the statistical results of the TOP500 list, more and more supercomputers are using the open-source Linux operating system. It is not only because of the low cost in software (Linux is an open-source operating system), training and porting, but also because Linux can provide a stable and expandable platform that allows users to continuously improve the system performance according to their own needs.

##### The Cluster Network Security

To facilitate the management of the cluster, the administrator usually connects the office network and the cluster management network. However, since the office network is usually interconnected with the Internet, this 'fake physical isolation' connection is at a considerable risk of security. Despite of the convenience, the connection between the office network and the cluster management network also opens a gateway for the hackers to attack. The attacks on the cluster system mainly include application-level attacks, network-level attacks, and infrastructure-level attacks.

A variety of network threats from the outside can be prevented by installing the firewall and VPN servers in the front-end of the cluster system. The architecture with the protective functions is shown in Fig. 2.10.

- (1) The firewall. The current firewall products on the market can be divided into two categories, the hardware firewall and the software firewall. The software firewall is mostly based on PC architecture, and may adopt an optimized OS as its operating platform. The features of the software firewall include good scalability, strong adaptability, easy to upgrade, and far less cost than the hardware-based firewall.

- Most hardware firewalls are based on ASIC (Application-Specific Integrated Circuit). The advantages over the software firewall include faster speed, better

Fig. 2.10 The cluster system protection

![Image](/assets/img/posts/2026-03-13-The_Student_Supercomputer_Challenge_Guide_Chapter_2_artifacts\image_000010_43083f2ad619445539ac43d76943a93febeb62ed334e79d022a489a13cd21e32.png)

- stability, and higher safety factor. However, the hardware option is more expensive, less scalable, and more difficult to upgrade compared with the software firewall.

- Due to its high cost, the hardware firewall is used less frequently in practical systems. We usually use the open-source software firewall for the servers. There are many open-source firewalls based on the Linux operating system, such as Zentyal, pfSense, IPFire, SmoothWall, ClearOS, and Iptables. Specifically, Iptables, which is embedded in the Linux kernel, is a powerful firewall that supports configuration of various rules. With a proper configuration of the IP rules, the system security can be significantly improved. In addition, we can also configure the Iptables to prevent IP spoofing and Dos attack problems.

- (2) The VPN server. VPN (Virtual Private Network) is a remote access technology, which builds the private network based on the public network link. It is called a virtual network, mainly because in the entire VPN network, there is no end-to-end physical link used in the traditional private network. The VPN uses the logical network based on the network platforms (such as ATM, Frame Relay) provided by the public network service provider. The transmission of the user data is on the logical link. The VPN provides functions such as the encapsulation and encryption of the data, and authentication of the user identity across shared or public network, and is an extension of the traditional private network. OpenVPN is the most mature and most stable open-source VPN technology on Linux, and allows the establishment of VPN using the pre-set private keys, third-party certificates, or username/password for authentication. The OpenVPN uses the OpenSSL encryption library, and the SSLv3/TLSv1 agreement, and can support a number of different operating systems, such as Linux, xBSD, Mac OS X, and Windows 2000/XP.

- The VPN uses technologies such as the tunnel, encryption/decryption, key management, and the user/device authentication. OpenVPN has a number of inherent security features. For example, it can be run in the user space without modifying the kernel and network protocol stack; it gives up the root privileges and runs in the chroot mode after the initialization; and it uses the mlockall to prevent the exchange of the sensitive data to the disk.

#### 2.3.5.2 Build a 10-Tflops Cluster in the Homogeneous Mode

For a 10 Tflops or even more powerful CPU cluster, the heat dissipation, noise, and power supply are already beyond normal office environments, and need to be placed into specific computer room, which usually have state standards to follow.

##### The Compute Node

Taking the AVX-supported Intel SNB (Sandy-Bridge) platform into consideration, up to 40 2-way nodes are sufficient to provide the 10-Tflops computing performance. The detailed calculation is as follows:

The floating point capability per compute node equals:

2 (the CPU number per node)

x8 ( flops per clock cycle/the calculated cores number)

x8 (the cores number per CPU).

If the frequency is 2.0 GHz (the starting frequency of the E5-EP processors on the general SNB platform), the computing power of a 2-way compute node is more than 256 G flops, and 39 compute nodes can already provide the 10 Tflops (10 Tflops = 10 * 1000 G flops) peak performance.

As for the type of the compute nodes, we have the option of the rack-mount servers or the blade servers. If adopting the rack-mounted server, we will need at least 39U rack space to meet the needs of the computing capability; assuming we are building our system using 1U standard rack-mounted 2-way servers, we need at least 1 standard 42U cabinet. However, such a design leaves no space between the nodes, and the heat dissipation could become a problem. Therefore, if we can afford more space, we usually use at least two cabinets, and reserve the space for the switches and storages. We can also adopt the high-density twin rack-mounted server. Generally, 4 2-way nodes could be accommodated in a 2U space and the density is doubled so that all the compute nodes can be housed in 1 cabinet. If we choose the blade servers, taking the current popular high-density blade server Inspur NX5440 as an example, an 8U space can accommodate 20 2-way nodes. Hence, two NX5440 servers can meet all the requirement of the computing power.

##### The Management Node

Generally, the number of users for a 10-Tflops cluster should be less than 20. We do not need to use the expensive commercial cluster software, as the challenge for the monitoring, management, and job scheduling would be relatively small. Instead, we can use the free and open-source software to meet the demands, such as openPBS. As mentioned above, due to the relatively small pressure on the hardware, the management node can share the same physical machine with the login node.

##### The Login Node

The login node and the management node can share the same machine, which can use a lower configuration than the compute node, such as a single rack-mounted server.

##### The Interconnect

When using the IB network, we need to consider different situations of the compute nodes. If adopting the rack-mounted nodes, we need to use a separate IB switch configuration. The number of the general 1U IB switch interfaces is configured to be 36. Over 36 interfaces will need to be configured with more switches. In this case, we need to consider whether the IB network needs the full line rate. If we need to do the full line rate, we also need to consider how to make the most economic interconnection among the servers and among the switches. If adopting the blade

nodes, we can directly interconnect the two blades through the IB network because the blade node generally has its own IB switch module.

When using the gigabit Ethernet as the computing network, we only need 1 48-port gigabit Ethernet switch. In the case of the using blade nodes (taking the NX5440 as an example), we can just interconnect the 10-gigabit uplink ports of the gigabit switch modules of 2 NX5440 blades.

##### The Management Network

We usually use the gigabit network as the management network, and the interconnection method is as mentioned above.

##### The Storage

For typical 10-Tflops HPC clusters, the I/O pressure is generally small, and we can take different approaches according to different situations. For example, if the output data is small, we can directly use the hard disk of the management node to do NFS data sharing and regularly back up the result data. On the other hand, if the output data is large, we recommend specialized storage devices, such as the storage servers or the disk arrays, and using the XFS file system.

#### 2.3.5.3 Build a 10-Tflops Cluster in the Heterogeneous Mode

In contrast to the homogeneous mode, it is simpler to build a 10-Tflops cluster in the heterogeneous mode. For the current 1-Tflops accelerator cards, such as NVIDIA Tesla K20M or Intel Xeon Phi, 10 accelerator cards are enough to provide the required 10-Tflops computing power. If combined with the computing power of the CPUs, the number of needed nodes will be significantly smaller.

When adopting the heterogeneous mode to build the clusters, we need to consider the application features and performance balance, as well as what kind of hardware architecture to use. For example, if the application demonstrates a lot of thread-level parallelism (e.g. using OPENMP heavily) and scales better within a single node, we should consider using high-density heterogeneous servers, with two CPUs and four GPU cards in a single compute node.

If the application scales better across multiple nodes, and we hope to do large-scale CPU computation as well as GPU computation, we can consider equipping each node with 2 CPUs and 1 GPU.

Even if adding only 1 GPU accelerator card to each node, to meet the same 10-Tflops performance requirements, we only need 12 compute nodes.

When building the 10-Tflops heterogeneous HPC system, the considerations about the compute nodes and network are the same as the CPU homogeneous cluster. However, as for the management node and login node, they must include the support for GPU management and scheduling. An example is the Inspur ClusterEngine, which can manage and monitor the GPUs and schedule the GPU jobs, and is a suitable solution for heterogeneous HPC systems.
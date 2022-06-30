# Title
Time and Energy-Efficient task offloading and Scheduling Algorithm in a 3 tier Fog Architecture

# About

Fog Computing paradigm is used to reduce the latency and computing cost of data produced by edge and IoT devices. The introduction of an intermediate fog layer between the device layer and the Cloud layer reduces cost and latency. This project involves simulating the fog layer and analysing the reduction in cost of system in terms of time and energy consumed.

# Project Report

More information please refer to the prject report attached and the Colab Notebook.

# Introduction
We propose a Time and energy efficient scheduler which allocates tasks from a particular
application based on characteristics like the energy consumer, total time taken, nature of the task
whether it is a critical or non critical task and its respective deadline. The task of finding minimal
cost and optimization is a NP hard problem due to the following reasons:

1. A single model cost and energy optimization problem is not a NP hard problem but
modeling our cost based on just these 2 factors would not be sufficient. We need to take
into consideration factors which contribute to time and energy like the energy consumed
during data transmission, the energy consumed for data download, the time taken for data
transfer and the objective of minimizing these variables for each individual layer i.e IOT,
FOG and cloud layer. Thus this is a multi-objective optimization problem.

2. We need data at a granular level and a method of calculating these parameters at each and
every level. Only then a comprehensive analysis of the energy and time factor can be
feasible for developing a real life solution. Thus we need a simulation procedure which
provides us the tools to simulate and collect data at each and every step.

## 1.1 Processing sites
There are three available resource allocation options in the system:
* Local processing, in the Iot device: The scheduler can choose to keep the task in the IoT device and execute it locally, in it's own core.

* Local processing, in the MEC server: MEC servers are local servers that bring Cloud services close to the final user and it's main objective is to lower latency for the final user (Yu, 2016). Instead of offloading a task to the Cloud (higher latency) the task can the offloading to a smaller server (MEC server) that has processing capabilities much higher than the IoT device, but has lower latency than the Cloud. 

* Remote processing, in the Cloud's Data Centers: Data Centers are distant-centrilized processing clusters with high procesisng capabilities and virtual infinite resources. It's and available offloading option, but may add a lot of latency to the IoT application.

## 1.2 Types of tasks
The system has two different types of tasks, critical tasks and non-critical tasks. The critical tasks have deadline stablished on their creation, with means that they have to finish execution before reaching the deadline, otherwise the task will be automatically cancelled. The non-critical tasks don't have a deadline, so they won't be cancelled if processing takes too long.

## 1.3 Application and it's characteristics
When running the simulator we have to choose an application first. The application has a set of characteristics that must be configured in the simulator por it's propor execution. The characteristcs we have to set are:

* **Task generation rate**: Define how long will it take for tasks from the application to be created from one another.
* **Data size entry**: Define the size of the data the task needs for processing. This data size has impact in the data transmissions across the system adding energy and time costs for the task, depending on the allocation option.
* **Results size**: Define the size of the results a task delivers after finishing processing. For instance, if a task is allocated in a MEC server, it's results must be sent back to the origin, that is, the IoT device (user) that created the task.
* **Computacional load**: It's assumed that the user creates cyclical tasks and knows the number of CPU cycles a task have. The computacional load is used mainly to calculed the time needed for execution. 
* **Deadline**: Define the deadline for critical tasks. If a critical task is created, data transmissions (data entry and results) and processing must be lower than the deadline, otherwise the task will be cancelled.

Below there are two examples of applications. Application 1 is considered of high processing cost, while Application 2 is considered of low processing cost.

![Examples of applications](images/Applications.PNG)

# 2 Architecture

The three layer architecture uses two different communication technologies. The IoT device layer communicates with the MEC layer with 5G and the MEC layer communicates with the Cloud layer with fiber optics.

![Architecture of the system](images/architecture.png)

The MEC servers are local servers placed close to the final user (IoT devices in this case) which provide lower latency when running tasks origined in those IoT devices, if compared to the Cloud latency. Even with more latency added to the application when tasks run in the MEC server or in the Cloud, it could be a better choice then running locally in the IoT's device core, because the executing time could be very high in the user's device. 

It is expected that the IoT devices move around the network, that's why a wireless connection is required between the bottom layer and the intermediate layer, so that offloading of data and task's source code can be performed.

## 2.1 Hardware description

The hardware that is describe below is used in the simulator, each element with it's corresponding characteristics.

* **IoT devices**: Each IoT device is assumed to be an Arduino Mega 2560. THe Arduino Mega 2500 has an ATmega2560 microcontroller with one 16MHz core. The available configurable frequencies for the microcontroller are 16 MHz, 8 MHz, 4MHz, 2 MHz and 1 MHz. The associated core voltages for each of the frequencies are, respectively, 5V, 4V, 2.7V, 2.3V and 1.8V. So, running a task with clock set to 2 MHz will make the voltage core be set to 2.3V.

* **MEC server**: MEC servers in the intermeadiate layer are composed of 5 Raspberry Pi4 Model B boards each. Raspberry Pi 4 ModelB is  equipped with a Quad-core Cortex-A72 1.5GHZ (ARMv8) 64-bit, summing a total of 20 CPUs per MEC server. Each CPU core has operating frequencies of 1500 MHz, 1000 MHz, 750 MHz and 600 MHz; the corresponding supply voltages are 1.2 V, 1V, 0.825 V and 0.8 V.

* **Data Center**: For the Cloud Computing layer were chosen Intel Xeon Cascade Lake processors of 2.8 GHz per CPU, reaching up to 3.9 GHz with Turbo Boost on. These processors can be found in some configurations on the Google Cloud.

* **Data connections**: It was established that both 5G and fiber optic communications could reach speeds of up to 1Gbps. For both, latency is set at 5ms. So the transfer time of a piece of data is the same in both cases. What differs is the energy consumption.

# 3 Calculation total task cost
The scheduling algorithm (presented in Chapter 4.) implements a cost model that evaluates the cost of processing a task locally in the IoT device, locally in the MEC server ou remotelly in the Cloud. The cost model and the equations used to calculate energy consume and elapsed time are shown in the following sections.

## 3.1 Time equations
To calculate the execution time in a CPU core, we have to know the total number of CPU cycles the task have (*CT*) and the core's operating frequency (*f*).

![Execution time in a CPU node](images/TIMEexecution.png) (Tanenbaum; Austin, 2012)

Also, if we have data transmissions we need the following equations:

![Elapsed time for data entry plus source code transfer from the IoT device to the MEC server](images/TIMEup.PNG) (Yu; Wang; Guo, 2018)

![Elapsed time for results transfer from the MEC server to the IoT device](images/TIMEdown.PNG) 

The variable ri(h) is the transfer rate between two layers of the architecture. Note that for IoT to MEC and MEC to IoT transfers, we use the "up" equation to send the task's entry data and source code from the IoT device to the MEC server and then the "down" equation to send the results from the MEC server back to the IoT device. If the processing is made in the Cloud, then we have an additional transfer from MEC to Cloud (entry data and source source) and then from Cloud to MEC (results). That's why running tasks in the Cloud adds more latency.

## 3.2 Energy equations
When we have some load to be processed in a CPU core the power consumed is equal to:

![Power equation for CPU](images/POWERdynamic.png) 

For the dynamic power equation, *C* stands for capacitance, *V* for volts (CPU core tension) and *f* from frequency There are other components that could be used to make the total power more acurate, like leak power and short circuit power, aside form the load power (dynamic power). These two represent a little percentage of the total power, and are disregarded here (Burd; Brodersen, 1996).

Given the power consumed by a CPU core the total energy consumed can be calculated as:

![Energy equation for CPU](images/ENERGYconsumed.png)

## 3.3 DVFS
The DVFS (Dynamic Voltage and Frequency Scaling) technique is used in order to alter the energy consumed by the CPU cores of both IoT devices and MEC servers during execution time (Sarangi; Goel; Singh, 2018). The TEMS algorithm calculates the final costs of all options of operating frequency and core voltage and chooses the best fitting option, i. e. the pair frequency-voltage that provides the minimum calculated cost for the system.

Altering the frequency and voltage in the power equation presented in Section 3.2, we have different power levels. With a lower frequency the power and energy consumed will be smaller, but execution time may be too long. **That is a trade-off the scheduling algorithm deals with**.

To make things clearer, look at the image below and see how pair of frequency and voltage can alter the power of a CPU core.

![DVFS for a Intel Pentium M processor of 1.6GHz](images/DVFScore.PNG) (INTEL, 2004)

## 3.4 Cost model equations by task and for the system
Using the equations shown before we can calculate the total elapsed time and total energy consumed by **one** task if allocated to the IoT device itself:

![Task cost for the IoT device](images/TaskCostEquantion.PNG) 

The same goes for the cost in the MEC server and in the Cloud. In the equation above UlocalE and UlocalT represent, respectively, the energy and time coefficients used to make either energy or time more costly for this allocation option. 

To find the minimum cost in the system for the task we compare the costs of all allocation options and choose the smallest one:

![Cost of a single task](images/COSTsingleTask.PNG)

In this equation the allocation option that yields the lowest cost per task is chosen. This process is made for all taks in the system. The alpha, beta and gama variable are priority variables, that can be set to prioritize one allocation option or another. A high value in a priority variable makes the corresponding allocation option more costly, which means it has lesser changes of being chosen. To make one particular allocation option more appealing (low cost) a small priority variable may be chosen.

![Systems cost](images/COSTsystem.PNG)


## 4 Dependencies
This project uses javatuples-1.2.jar. This library is included in the "libraries" folder.

# References:
1. Yu, H.; Wang, Q.; Guo, S. Energy-efficient task offloading and resource scheduling for
mobile edge computing. In: 2018 IEEE International Conference on Networking,
Architecture and Storage (NAS). [S.l.: s.n.], 2018. p. 1–4.
2. Offloading in Mobile Edge Computing: Task Allocation and Computational Frequency
Scaling," by T. Q. Dinh, J. Tang, Q. D. La and T. Q. S. Quek. IEEE Transactions on
Communications, vol. 65, no. 8, pp. 3571-3584, Aug. 2017.
3. Guevara, Judy C., and Nelson L. S. da Fonseca. “Task Scheduling in Cloud-Fog
Computing Systems.” Peer-To-Peer Networking and Applications, vol. 14, no. 2, Jan.
2021, pp. 962–77, https://doi.org/10.1007/s12083-020-01051-9.Aazam, Mohammad, et
al. “Offloading in Fog Computing for IoT: Review, Enabling Technologies, and Research
Opportunities.” Future Generation Computer Systems, vol. 87, Oct. 2018, pp. 278–289,
10.1016/j.future.2018.04.057. Accessed 8 Apr. 2020.

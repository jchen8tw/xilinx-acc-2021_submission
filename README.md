# Xilinx ACC 2021 Submission
EXTREME TRADING SOLUTION: Accelerated Quantum-Inspired Algorithm on Accelerated Algorithmic Trading (AAT) Framework

## 1-1 Project Abstraction
Quantum-Inspired Algorithm has been used to optimize combinatorial problems in place of Universal Quantum Computation. We design and implemented two Quantum-Inspired  Trading Strategies for currency arbitrage, Simulated Bifurcation Machine (SBM), and Simulated Quantum Annealing SQA. Both are implemented and validated on Xilinx Algorithmic Trading (AAT) framework. This is a fully-featured open-source reference design for trading applications.

## 1-2 Data Generation

## 1-3 HW Synthesis & SW Compiling

Development Environment and Sources:
* Operation System of Ubuntu 20.04.2 LTS
* Xilinx Vitis Software Platform 2021.1
* Xilinx Accelerated Algorithmic Trading reference design package Q2 (UG1067 v1.1, July 2 2021)
* SBM/SQA design sources in sbm/sqa directory

Build flow of AAT and SBM/SQA design sources
* Overwrite the origional AAT Q2 files in `../Accelerated_Algorithmic_Trading/hw/pricingEngine` with the files in  
`sbm/src/hw/pricingEngine` or `sqa/src/hw/pricingEngine`.

<img src="https://user-images.githubusercontent.com/11850122/157410864-081a295e-f9a1-45fb-a2c7-c7d8faa9d1ca.png" width=65%>

Settings in `~/.bashrc`:

    source /opt/Xilinx/Vitis/2021.1/settings64.sh
    source /opt/xilinx/xrt/setup.sh
    export PLATFORM_REPO_PATHS='/opt/xilinx/platforms'
    export LM_LICENSE_FILE="~/Xilinx.lic"
    export XILINX_PLATFORM='xilinx_u50_gen3x16_xdma_201920_3'
    export DEVICE=${PLATFORM_REPO_PATHS}/${XILINX_PLATFORM}/${XILINX_PLATFORM}.xpfm
    export DM_MODE=DMA
    
 Build Hardware & Software:

    $ cd ../Accelerated_Algorithmic_Trading/build
    $ make clean
    $ ./buildall.sh

Build Software:

    $ cd ../Accelerated_Algorithmic_Trading/sw/applications/aat/aat_shell_exe
    $ make all
    
## 1-4 Test Flow
Preparation:
* A local host installed with a Xilinx Avelon U50 accelrator
* A remote host installed with a Broadcom BCM957711A 10Gb x 2 SFP port card and PCAP test files
* A QSFPx1-to-SFPx4 connection cable
* An AAT `demo_setup.cfg` and SFP network setting files in submitted configuration directory

Supplementary of following test instructions:
* Our own SPF port names are `enp3s0f0` and `enp3s0f1`. You should replace `enp3s0f0` and `enp3s0f1` with your own SPF port names in `settingNetwork_sf0.sh` and `settingNetwork_sf1.sh`.
* The used `enp3s0f1` parameter in Linux TCPreplay command terminal#2 is also should be replaced with your own SPF port name, which is corresponded to the port to send PCAP files.
* The `./sample/aat.u50_xdma.xclbin` can be another path and xclbin naming, but it should be the built xcblin file from AAT & SBM or AAT & SQA designs    
* The AAT Q2 default `support/demo_setup.cfg` should be replace with submitted `configuration/demo_setup.cfg`

We refer the network configuration used by the Xilinx verification team.

<img src="https://user-images.githubusercontent.com/11850122/157402051-63b60368-00ba-4987-aada-0ed15f9da004.png" width=50%>

Run AAT shell terminal#0 on U50 local host.

    sudo reboot (if needed to clean U50 setting)
    cd ../Accelerated_Algorithmic_Trading/build
    vim support/demo_setup.cfg (if needed to change demo_setup.cfg setting)
    ./aat_shell_exe
    download ./sample/aat.u50_xdma.xclbin
    run support/demo_setup.cfg
    datamover threadstart
    udpip0 getstatus
    
Run Linux Netcat command terminal#1 on remote host and get AAT output from local host.

    cd ../Network_setting/
    sudo ./settingNetwork_sf0.sh
    sudo ./execFrom_sf0.sh ping -w 5 192.168.20.200 (optional test)
    sudo ./execFrom_sf0.sh nc -n -l 192.168.20.100 12345 -v
    
If Linux Netcat command terminal#1 has not shown connected IP & Port message from local host, run OrderEntry reconnection and get its status on U50 local host terminal#0.

    orderentry reconnect
    orderentry getstatus

On U50 local host terminal#0, connection established should be shown "true" and connection status should be shown "SUCCESS”.

<img src="https://user-images.githubusercontent.com/11850122/155680914-ad137fe7-37af-4048-a270-ee72ed263c0e.png" width=45%>

Run Linux TCPreplay command terminal#2 to send AAT input PCAP files from reomte host.

    cd ../Network_setting/
    sudo ./settingNetwork_sf1.sh
    sudo ./execFrom_sf1.sh ping -w 5 192.168.50.101 (optional test)
    sudo ./execFrom_sf1.sh tcpreplay --intf1=enp3s0f1 --pps=2 --stats=1 ../Accelerated_Algorithmic_Trading/build/sample/cme_input_arb.pcap

## 2-1 Currency Arbitrage QUBO Formulation
Before we can apply SBM/SQA to find optimal arbitrage opportunities, we need to transform the problem into a quadratic unconstrained binary optimization (QUBO) form. The methodology is introduced in

https://github.com/bol-edu/xilinx-acc-2021_submission/tree/main/qubo_formulation

## 2-2 SBM Design & Implementation
Simulated bifurcation is a Quantum-Inspired heuristic algorithm that approximates the solutions to Ising-model problem formulations. The algorithm can yield high-quality solutions under fewer time steps than the traditional simulated annealing method. Furthermore, the matrix-vector multiplications in the equations can be simplified and parallelized easily, opening the possibility for high frequency trading. In this project, SBM solves a currency arbitrage problem of 5 currencies and 9 exchange pairs in under 7 microseconds, demonstrating its speed and modularity. SBM is integrated into the `pricingEngine` of the AAT framework, which processes the market data and decides the orders to place.

https://github.com/bol-edu/xilinx-acc-2021_submission/tree/main/sbm

## 2-3 SQA Design & Implementation
The complexity of the original SQA algorithm is O(M*N*N) in time and O(N*N) in space, where N and M are the numbers of spins and trotters. This work applies some optimization techniques to boost the FPGA performance. For detail, please refer to 

https://github.com/bol-edu/xilinx-acc-2021_submission/tree/main/sqa

## 3 Features and Benefits
Quantum-Inspired Trading Strategies:
* Leverage HLS to develop trading strategies with readability and maintainability
* Customizable Quantum-Inspired trading strategies
* Parallel computing on FPGA hardware
* Built on trading industry-standard of CME and FIX

Accelerated Algorithmic Trading (AAT) Framework:
* Full implementation in HLS
* Designed for software engineers
* Source code provided
* Integrated with Vitis™
* Runs out of the box on Xilinx Alveo™
* Supported on Alveo™ U250, U50
* Supports XDMA and host memory bridge-based

Quantum-Inspired trading strategies can be executed on CPUs/GPUs but incur additional latency particularly from traversing the PCIe bus. FPGA based trading strategies can significantly lower latency, but typically need large teams of hardware experts and long design cycles.
Xilinx AAT reference design can provide all the infrastructure required to create a Quantum-Inspired trading application on the FPGA using Xilinx Vitis™ unified platform, and standard Xilinx shells. The design is written in HLS, and all the source code is provided. The design is modular, allowing easily replacing trading strategy IP blocks in the reference design with needed strategy.

<img src="https://user-images.githubusercontent.com/11850122/156995074-9c4ba66d-568a-41d6-979d-5c7943572f82.png" width=120%>

AAT Shell Enhancement for HW Debugging:
* Status/Debugging registers for SQA/SBM modules
* Add timestamp and time pause functions in both shell mode and run script mode

https://github.com/bol-edu/xilinx-acc-2021_submission/tree/main/aat_shell_enhancement


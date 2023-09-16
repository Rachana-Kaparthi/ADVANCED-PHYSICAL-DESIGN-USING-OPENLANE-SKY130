# ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130
[Day 1- Inception of open-source EDA,Openlane and Sky130 PDK](#day-1---inception-of-open-source-edaopenlane-and-sky130-pdk)  

[Day 2 - Good flooplan vs Bad Floorplan and introduction to library cells](#day-2---good-flooplan-vs-bad-floorplan-and-introduction-to-library-cells)  

[Day 3 - Design Library cell using Magic Layout and NG Spice Characterisation](#day-3---design-library-cell-using-magic-layout-and-ng-spice-characterisation)  

[Day 4 - Timing Analysis using OpenSTA]()

[References](#references)


## Day 1 - Inception of open-source EDA,Openlane and Sky130 PDK
<details>
<summary>Installation of Required Tools</summary>  
<details>  
<summary>OpenLane </summary>
  
OpenLane is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, CVC, SPEF-Extractor, KLayout and a number of custom scripts for design exploration and optimization. It also provides a number of custom scripts for design exploration and optimization.  

Before installing Openlane, we should first install its dependencies:
  
```
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```
Docker Installation:

```
# Remove old installations
sudo apt-get remove docker docker-engine docker.io containerd runc

# Installation of requirements
sudo apt-get install \
   ca-certificates \
   curl \
   gnupg \
   lsb-release

# Add the keyrings of docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add the package repository
echo \
   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package repository
sudo apt-get update

# Install Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Check for installation
sudo docker run hello-world

sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot

# After reboot
docker run hello-world
```
Now download Openlane from Github:
```
git clone --depth 1 https://github.com/The-OpenROAD-Project/OpenLane.git
cd OpenLane/
make
make test
cd /home/rachana/OpenLane/designs/ci
cp -r * ../
```
</details>  
<details>   
<summary>OpenSTA</summary>

Use the following commands to checkout the git repository and build the OpenSTA library and excutable.
```
#installing dependencies for OpenSTA
sudo apt-get install cmake clang gcc tcl swig bison flex

#installing OpenSTA
git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
cd OpenSTA
mkdir build
cd build
cmake ..
make
```
</details>
<details>
<summary>Magic</summary>  

Use the below commands for installing Magic.  

```
sudo apt-get install m4
sudo apt-get install tcsh
sudo apt-get install csh
sudo apt-get install libx11-dev
sudo apt-get install tcl-dev tk-dev
sudo apt-get install libcairo2-dev
sudo apt-get install mesa-common-dev libglu1-mesa-dev
sudo apt-get install libncurses-dev
git clone https://github.com/RTimothyEdwards/magic
cd magic
./configure
make
sudo make install
```
</details>  
<details>  
 <summary> NG Spice   </summary>
Download the tarball from [here](https://sourceforge.net/projects/ngspice/files/) to a local directory and unpack it using the following commands:

```
tar -zxvf ngspice-40.tar.gz
cd ngspice-40
mkdir release
cd release
../configure  --with-x --with-readline=yes --disable-debug
make
sudo make install

```
  </details>

</details>
<details>
<summary>SoC Design and Openlane</summary>
Application Specific Integrated Circuit(ASIC) consists of 3 main parts:  
  
  - RTL IP's
  - EDA Tools
  - PDK Data  
  
In short, it can be implemented as below:  
  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/asic_elements.png)

The main objective of ASIC is to convert the code from RTL level to GDSII which is used for final layout process.GDSII stream format (GDSII), is a binary database file format which is the de facto industry standard for Electronic Design Automation data exchange of integrated circuit or IC layout artwork.  

**Simplified  RTL to GDSII Flow**
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/rtl_to_gdsii.png)
*Synthesis:*

Convert your RTL code into a gate-level netlist using synthesis tools such as Yosys. This step generates a logical representation of your design using standard cells from a library.  

*Floor Planning:*

Define the physical layout of the chip, including the placement of functional blocks, I/O pads, and power grid distribution. This step helps determine the chip's overall size and shape.  

*Power Planning:*

Implement the power distribution network to provide stable power to all parts of the design while minimizing voltage drop. Tools like OpenSTA can be used for static timing analysis to ensure proper power distribution.  
Placement:

Place the synthesized logical cells onto the chip's floorplan. Tools like RePLace or Graywolf can be used for placement.  
Placement is usually done in two steps:  

- Global Placement
- Detailed PLacement

*Clock Tree Synthesis (CTS):*

Generate a clock distribution network that ensures clock signals reach all parts of the design with minimal skew. Typically, OpenLane's TritonCTS is used for this purpose.It ususally takes the shape of a tree..  

*Routing:*

Create the physical interconnections (metal layers) between the placed cells while adhering to design rules. This step is performed using a router like FastRoute or TritonRoute.  
Metal Layer form a routing grid which is huge, hence we use divide and conquer methodology for routing grid.  
Global Routing: Generates routing grids  
Detailed Routing: Uses the routing guides to implement the actual wiring  

*Sign Off:*  

This includes physical and Timing verifications.  
- *Physical verifications:*
  - *Design Rule Checking (DRC):*
    - Verify that the chip layout adheres to the manufacturing process's design rules. DRC tools like Magic or KLayout are commonly used for this purpose.
  - *Layout vs. Schematic (LVS) Check:*
    - Ensure that the final layout matches the original schematic. LVS tools like Netgen or Calibre are used to compare the netlist extracted from the layout with the synthesized netlist.
- *Timing verifications:*
  - *Static Timing Analysis (STA):*
    - Analyze the timing characteristics of your design to ensure that all setup and hold time requirements are met. OpenSTA is commonly used in the OpenLane flow for STA.
      
**Opensource ASIC flow**  

The OpenLANE flow utilizes tools mainly from the OpenROAD, YosysHQ, and Open Circuit Design projects. The way those tools are used, augmented by a number of other custom tools and scripts, defines the methodology of the flow.
OpenLANE supports two main use cases-  

- First, It can be used to harden designs from their RTL HDL models obtaining what we will refer to as soft macros
- The second use case is integrating macros into a complete chip.
  
To demonstrate its capabilities, OpenLANE has been used to successfully tape out a family of RISC-V based SoCs called striVe.

Below figure demonstrates the Openlane ASIC flow-  

![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/openlane_asic_flow.png)  

 Below is a summarized breakdown of the stages seen in the figure:  
 
 A. RTL Synthesis and STA -The design is synthesized into a gate-level netlist using yosys and static timing analysis is performed on the resulting netlist using OpenSTA.  
 
 B. Insertion of DFT structures -An open-source Design For Testability (DFT) toolchain, Fault, can optionally be used to modify the netlist, inserting scan chains and the necessary IO ports to scan and test the design after fabrication.  
 
 C. Physical Implementation -Most of the tools in this stage are used from within the OpenROAD application in combination with other tools, some of them are custom and based on the OpenDB infrastructure,while others are indpendent.  
 
  D. Post-routing Evaluation of Results -DRC and LVS are then performed using magic and netgen .Antenna checking is performed by either OpenROAD’s ARC (Antenna Rule Checker) or using magic.  
 </details>  
 
<details>
  <summary>Open-Source EDA Tools</summary>

OpenLANE utilises a variety of opensource tools in the execution of the ASIC flow:  

Task | Tools
------------- | -------------
RTL Synthesis & Technology Mapping | [yosys](https://github.com/YosysHQ/yosys), abc
Floorplan & PDN | init_fp, ioPlacer, pdn and tapcell
Placement | RePLace, Resizer, OpenPhySyn & OpenDP
Static Timing Analysis | [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
Clock Tree Synthesis | [TritonCTS](https://github.com/The-OpenROAD-Project/OpenLane)
Routing | FastRoute and [TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute) 
SPEF Extraction | [SPEF-Extractor](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
DRC Checks, GDSII Streaming out | [Magic](https://github.com/RTimothyEdwards/magic), [Klayout](https://github.com/KLayout/klayout)
LVS check | [Netgen](https://github.com/RTimothyEdwards/netgen)
Circuit validity checker | [CVC](https://github.com/d-m-bailey/cvc) 

**Steps to synthesis in OpenLane:**  
```
cd ~/OpenLane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/run_synthesis.png)  

After we run synthesis command, new folder named 'runs' will be created in the picorv32a directory where we find the simulation results, logs etc related to picorv32a synthesis.
Netlist of picorv32 can be seen here-  
```
cd /home/rachana/OpenLane/designs/picorv32a/runs/RUN_2023.09.09_15.50.10/results/synthesis
gedit picorv32a.v
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/picorv32a_netlist.png)  

Reports can be seen here:
```
cd /home/rachana/OpenLane/designs/picorv32a/runs/RUN_2023.09.09_15.50.10/reports/synthesis
gedit 1-synthesis.AREA_0.stat.rpt
```
Synthesis report:  
```
=== picorv32 ===

   Number of wires:               9824
   Number of wire bits:          10206
   Number of public wires:        1512
   Number of public wire bits:    1894
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:              10104
     sky130_fd_sc_hd__a2111o_2       2
     sky130_fd_sc_hd__a211o_2      101
     sky130_fd_sc_hd__a211oi_2       4
     sky130_fd_sc_hd__a21bo_2       19
     sky130_fd_sc_hd__a21boi_2       7
     sky130_fd_sc_hd__a21o_2       414
     sky130_fd_sc_hd__a21oi_2      127
     sky130_fd_sc_hd__a221o_2       65
     sky130_fd_sc_hd__a221oi_2       1
     sky130_fd_sc_hd__a22o_2       197
     sky130_fd_sc_hd__a22oi_2        2
     sky130_fd_sc_hd__a2bb2o_2      16
     sky130_fd_sc_hd__a311o_2       38
     sky130_fd_sc_hd__a31o_2        90
     sky130_fd_sc_hd__a31oi_2       10
     sky130_fd_sc_hd__a32o_2        89
     sky130_fd_sc_hd__a41o_2         2
     sky130_fd_sc_hd__and2_2       283
     sky130_fd_sc_hd__and2b_2       32
     sky130_fd_sc_hd__and3_2        77
     sky130_fd_sc_hd__and3b_2       76
     sky130_fd_sc_hd__and4_2        46
     sky130_fd_sc_hd__and4b_2        6
     sky130_fd_sc_hd__and4bb_2       3
     sky130_fd_sc_hd__buf_1       2735
     sky130_fd_sc_hd__buf_2         16
     sky130_fd_sc_hd__conb_1       106
     sky130_fd_sc_hd__dfxtp_2     1596
     sky130_fd_sc_hd__inv_2         83
     sky130_fd_sc_hd__mux2_2      1817
     sky130_fd_sc_hd__mux4_2       323
     sky130_fd_sc_hd__nand2_2      250
     sky130_fd_sc_hd__nand2b_2       2
     sky130_fd_sc_hd__nand3_2       18
     sky130_fd_sc_hd__nand3b_2       3
     sky130_fd_sc_hd__nand4_2        2
     sky130_fd_sc_hd__nor2_2       185
     sky130_fd_sc_hd__nor3_2        11
     sky130_fd_sc_hd__nor3b_2        3
     sky130_fd_sc_hd__nor4_2         4
     sky130_fd_sc_hd__nor4b_2        3
     sky130_fd_sc_hd__o2111a_2       1
     sky130_fd_sc_hd__o211a_2      224
     sky130_fd_sc_hd__o211ai_2       6
     sky130_fd_sc_hd__o21a_2       154
     sky130_fd_sc_hd__o21ai_2       94
     sky130_fd_sc_hd__o21ba_2       15
     sky130_fd_sc_hd__o21bai_2       3
     sky130_fd_sc_hd__o221a_2       19
     sky130_fd_sc_hd__o221ai_2       1
     sky130_fd_sc_hd__o22a_2        26
     sky130_fd_sc_hd__o22ai_2        1
     sky130_fd_sc_hd__o2bb2a_2       7
     sky130_fd_sc_hd__o311a_2       31
     sky130_fd_sc_hd__o311ai_2       2
     sky130_fd_sc_hd__o31a_2        21
     sky130_fd_sc_hd__o31ai_2        2
     sky130_fd_sc_hd__o32a_2        14
     sky130_fd_sc_hd__o41a_2         1
     sky130_fd_sc_hd__or2_2        337
     sky130_fd_sc_hd__or2b_2        20
     sky130_fd_sc_hd__or3_2        102
     sky130_fd_sc_hd__or3b_2        17
     sky130_fd_sc_hd__or4_2         29
     sky130_fd_sc_hd__or4b_2         6
     sky130_fd_sc_hd__xnor2_2       78
     sky130_fd_sc_hd__xor2_2        29

   Chip area for module '\picorv32': 102957.494400
```
Flop ratio = (No.of D flipflops)/(Total no.of cells) =1596/10104 = 0.1579

</details>

## Day 2 - Good flooplan vs Bad Floorplan and introduction to library cells  

<details>
  <summary>Chip Floor Planning considerations  </summary>  

  There are two important parameters when it comes to floorplanning namely, Utilisation Factor and Aspect Ratio. 
  
*Utilisation Factor:*  

 - The ratio of area occupied by the cells in the netlist to the total area of the core
 - It is better to have a utilization Factor of 0.5 to 0.6 to accomodate any extra logic later on.
   
*Aspect Ratio:*

 - The ratio of height of a die to its width is defined as Aspect Ratio.
 -  Aspect ratio of 1 signifies that the die is of square shape and any other value other than 1 signifies that the die is rectangular shape.
   
**Floor planning**

The arrangement of IP's on a chip is referred to as floor planning.

*Pre-placed cells:*  

Whenever there is a complex combinational circuit, it can be divided into multiple sets of black boxes with inputs and outputs declared and placed on the core at fixed positions. pre-placed cells refer to specific logic blocks, memory elements, or other functional units that are fixed in their positions on the chip's layout during the initial stages of design and cannot be moved to a different position later on. These cells are placed manually by the chip designer or through automated tools. Since these IP's are placed before automated Placement and Routing, these are reffered to as Pre-placed cells.  

*Decoupling capacitors:*  

Pre-placed cells must then be surrounded with decoupling capacitors (decaps). The resistances and capacitances associated with long wire lengths can cause the power supply voltage to drop significantly before reaching the logic circuits. This can lead to the signal value entering into the undefined region, outside the noise margin range. Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.   
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/decap.png)  
In the figure Blocks A, B and C are preplaced cells surrounded by Decoupling capacitors.  

*Power Planning:*  
When there is a single power supply and ground on the chip, following effects may occur:

- Voltage droop, also known as voltage sag or voltage drop, refers to a temporary reduction in the power supply voltage at a specific point on the chip when a high current demand occurs. This condition arises when several blocks or cells try to draw power at the same time. 
- Ground Bump is a transient effect that can occur during the operation of the circuit where the voltage level of the ground (GND) signal temporarily rises or "bounces" above its reference voltage due to the switching of digital logic gates or other high-current activities. This condition arises when several blocks or cells try to dissipate power at the same time.

If voltage drops below Noise margin level in case of Voltage droops or voltage rises above Noise margin level in case of ground bumps then this results in undesired states.To mitigate this issue power supply and Ground ports are placed as grid of horizontal and vertical tracks so that the blocks draw power or dissipate power to the nearest power supply/ground intersection points.  

![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/power_supply.png)  

*Pin Placement:*  

The netlist defines connectivity between logic gates. The place between the core and die is utilised for placing pins. The connectivity information coded in either VHDL or Verilog is used to determine the position of I/O pads of various pins. The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay.

</details>
<details>
  <summary>Floorplan run on OpenLANE & view of the layout in Magic</summary>  
 
* Floorplan envrionment variables or switches:

1. ```FP_CORE_UTIL``` - floorplan core utilisation
2. ```FP_ASPECT_RATIO``` - floorplan aspect ratio
3. ```FP_CORE_MARGIN``` - Core to die margin area
4. ```FP_IO_MODE``` - defines pin configurations (1 = equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer
6. ```FP_CORE_HMETAL``` - horizontal metal layer

* Importance files in increasing priority order:
1. floorplan.tcl - System default envrionment variables
2. conifg.tcl
3. sky130A_sky130_fd_sc_hd_config.tcl
   
**Note: Usually, vertical metal layer and horizontal metal layer values will be 1 more than that specified in the files**
 
 To run the picorv32a floorplan in openLANE:
 ```
 run_floorplan
 ```
 ![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/run_floorplan.png)  
 
 Post the floorplan run, a .def file will have been created within the "results/floorplan" directory. We may review floorplan files by checking the "floorplan.tcl". The system defaults will have been overriden by switches set in "conifg.tcl" and further overriden by switches set in "sky130A_sky130_fd_sc_hd_config.tcl".  
 
To view the floorplan, Magic is invoked after moving to the results/floorplan directory:

```
magic -T /home/rachana/open_pdks/sky130/magic/sky130.tech lef read ../../tmp/merged.min.lef def read picorv32.def &
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/magic_output.png)  

One can zoom into Magic layout by selecting an area with left and right mouse clcik followed by pressing "z" key.  
Various components can be identified by using the ```what``` command in tkcon window after making a selection on the component
Zooming in also provides a view of decaps present in picorv32a chip:
![image](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/assets/140998470/0a807d5a-4919-4801-8ce8-10d82277aed6)
 
</details>  
<details>
  <summary>Library Binding and Placement</summary>  
  
**Placement Optimization**

The next step in the OpenLANE ASIC flow is placement. The synthesized netlist is the be placed on the floorplan. Placement is perfomed in 2 stages:

1. Global Placement: It finds optimal position for all cells which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length
2. Detailed Placement: It alters the position of cells post global placement so as to legalise them

Legalisation of cells is important from timing point of view. 

*Placement run on OpenLANE & view in Magic*

Congestion aware placement using RePIAce:
```
run_placement

```

![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/placement.png)   

The objective of placement is the convergence of overflow value. If overflow value progressively reduces during the placement run it implies that the design will converge and placement will be successful. Post placement, the design can be viewed on magic within ```results/placement``` directory:

```
magic -T /home/rachana/open_pdks/sky130/magic/sky130.tech lef read ../../tmp/merged.max.lef def read picorv32.def &

``` 
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/magic_placement.png)  

**Note:** Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN.  
 
</details>  
<details>
  <summary>Cell design and Characterization flows</summary>  
  Each cell that is placed on the layout is referred to as standard cell. Standard cells are pre-designed and pre-characterized logic gates, flip-flops, latches, and other digital components for which the definition is available in libraries. 
    
*Standard Cell Design Flow*

Standard cell design flow involves the following:
1. Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications 
2. Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power)
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

*Standard Cell Characterization Flow*

Characterization refers to the process of gathering and analyzing electrical and performance data for a specific cell or library element. The goal of characterization is to provide accurate and comprehensive information about how the cell behaves under various operating conditions. This information is essential for designing and optimizing digital circuits using these cells.  

A typical standard cell characterization flow includes the following steps:
1. Read in the models and tech files
2. Read extracted spice netlist
3. Recognise behaviour of the cell
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide necessary output capacitance loads
8. Provide necessary simulation commands
he opensource software called GUNA can be used for characterization. Steps 1-8 are fed into the GUNA software which generates timing, noise and power models.

*Timing threshold Definitions*

Timing defintion | Value
------------ | -------------
slew_low_rise_thr  | 20% value
slew_high_rise_thr |  80% value
slew_low_fall_thr | 20% value
slew_high_fall_thr | 80% value
in_rise_thr | 50% value
in_fall_thr | 50% value
out_rise_thr | 50% value
out_fall_thr | 50% value  
</details>

<details>
  <summary>Propagation Delay and Transition time</summary>

*Propagation Delay*

- Propagation delay refers to the time it takes for a change in an input signal to reach 50% of its final value to produce a corresponding change in the output signal to reach 50% of its final value of a digital circuit.

```
rise delay =  time(out_fall_thr) - time(in_rise_thr)
```
*Transition time*  

- Transition time refers to the time it takes for a digital signal to change its voltage level from one logic state (e.g., logic low or 0) to another logic state (e.g., logic high or 1) or vice versa.   
- Transition time is typically measured as the time interval between the moment when the signal voltage reaches a specific percentage (e.g., 10% to 90% or 20% to 80%) of its final value during a voltage transition and the moment when it reaches the opposite percentage during the subsequent transition.  

```
Fall transition time: time(slew_high_fall_thr) - time(slew_low_fall_thr)

Rise transition time: time(slew_high_rise_thr) - time(slew_low_rise_thr)
```

A poor choice of threshold points leads to neative delay value. Therefore a correct choice of thresholds is very important  

</details>  

## Day 3 - Design Library cell using Magic Layout and NG Spice Characterisation  

<details>
  <summary>CMOS Inverter NG Spice simulations</summary>  
  
**SPICE Deck creation & Simulation**

Before performing SPICE simulation, we have to create a SPICE Deck that contains the information about the following:

1. Component connectivity - how the components are connected
2. Component values - values of each component present in the circuit
3. Nodes - number of nodes and the elements connected between the nodes
4. Simulation type and parameters - type of simulation to be performed, say operating point, AC analysis or DC Analysis etc
5. Capacitance load - value of the capacitance connected at the load
6. Model description - model files that should be included in the simulation
7. Netlist description 

**Switching threshold[Vm]-**  

The point at which Vin=Vout is called switching threshold of CMOS.At this point both PMOS and NOMOS are in ON state which gives rise to a leakage current.  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/switching_threshold.png)  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/switching_threshold2.png)  

 **Steps to Gitclone vsdstdcelldesign**  
 
 The Magic layout of a CMOS inverter will be used so as to intergate the inverter with the picorv32a design. To do this, inverter magic file is sourced from [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign) by cloning it within the ```home/OpenLane``` directory as follows:
```
git clone https://github.com/nickson-jose/vsdstdcelldesign
```
This creates a vsdstdcelldesign named folder in the openlane directory.   
Now, we can view the layout of inverter in magic using the below command:  
```
magic -T libs/sky130A.tech sky130_inv.mag &
```
The layout shown in magic is as below:  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/magic_inv.png)    
To verify whether the layout is that of CMOS inverter, verification of P-diffusiona nd N-diffusion regions with Polysilicon can be observed:
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/magic_inv_tkcon.png)  

Other verification steps are to check drain and source connections. The drains of both PMOS and NMOS must be connected to output port (here, Y) and the sources of both must be connected to power supply VDD (here, VPWR).

**LEF or library exchange format:**
A format that tells us about cell boundaries, VDD and GND lines. It contains no info about the logic of circuit and is also used to protect the IP.  

Refer [here](https://github.com/nickson-jose/vsdstdcelldesign/blob/master/README.md#standard-cell-layout-design-in-magic) for step by step procedure of designing Standard cell layout in Magic.  

**SPICE extraction:**
Within the Magic environment, following commands are used in tkcon to achieve .mag to .spice extraction:
```
extract all
ext2spice cthresh 0 rethresh 0
ext2spice
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/spice_extraction_magic.png)  

*ext2spice* commands converts the ext file to spice netlist. cthreh and rthresh are the switches to extract all the parasitic resistance and capacitance. The extracted spice list has to be modified by changinf the scale, adding libraries,voltage sources,simulation commands as shown below to use ngspice to perform simulation:  

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/nshort.lib
.include ./libs/pshort.lib

//.subckt sky130_inv A Y VPWR VGND
M1000 Y A VGND VGND nshort_model.0 w=35 l=23
+  ad=1.44n pd=0.152m as=1.37n ps=0.148m
M1001 Y A VPWR VPWR pshort_model.0 w=37 l=23
+  ad=1.44n pd=0.152m as=1.52n ps=0.156m

VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

C0 Y VPWR 0.117f
C1 A VPWR 0.0774f
C2 Y A 0.0754f
C3 Y VGND 0.279f
C4 A VGND 0.45f
C5 VPWR VGND 0.781f
//.ends

.tran 1n 20n
.control
run
.endc
.end
```
Use the below command to view the output of the above netlist in NgSpice  
```
// to simulate the netlist file
> ngspice <filename>
// to plot the graph after simulation
> plot y vs time a
```
Below is the output of ngspice simulation:  

![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/ngspice_inv.png)  

The spikes in the output at switching points is due to low capacitance loads. This can be taken care of by editing the spice deck to increase the load capacitance value.

**Inverter Standard cell characterization**  

Four timing parameters are used to characterize the inverter standard cell:
*1. Rise transition:* Time taken for the output to rise from 20% of max value to 80% of max value   

From the graph,```Rise transition = (2.23843 - 2.17935) = 59.08ps```  

*2. Fall transition:* Time taken for the output to fall from 80% of max value to 20% of max value    
From the graph, ```Fall transition = (4.09291 - 4.05004) = 42.87ps```  

*3. Cell rise delay* = time(50% output rise) - time(50% input fall)  
From the graph, ```Cell rise delay = (2.20636 - 2.15) = 56.36ps```  

*4. Cell fall delay*  = time(50% output fall) - time(50% input rise)  
From the graph, ```Cell fall delay = (4.07479 - 4.05) = 24.79ps```  



</details>  
<details>
<summary>16-Mask CMOS Fabrication Process</summary>
A 16-mask CMOS (Complementary Metal-Oxide-Semiconductor) process is a manufacturing process technology that involves the use of 16 different masks or layers during the fabrication of integrated circuits. These masks are used to define various features and components on the semiconductor wafer, such as transistors, interconnects, and other essential elements. The number of masks used in a CMOS process can vary depending on the specific technology and the complexity of the integrated circuits being produced.  
Below are steps involved in 16-Mask CMOS Process-  

1. Substrate selection  
2. Creating active region for transistors
   - create Isolation between active region pockets by SiO2 and Si3N4 deposition followed by photolithography and etching which is termed as LOCOS(Local oxidation of Silicon) process.
     
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/locos.png)
The top layer that is present in the figure is of Silicon Nitrate(Si3N4) which is stripped using hot phosphoric acid.

3.N-well and P-well formation
  - Ion implanation by Boron for P-well and by Phosphorous for N-well formation.
  - ~200KeV of energy is required for Boron atoms to enter into P-substrate during ion implantation process for creating P well.
  - Same process is repeated with phosphorus atoms by applying ~400KeV(requires more energy as Phosphorus atoms are heavier than Boron) of energy for creating N well.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/well_creation.png)

  - High-temperature furnace process drives-in diffusion to establish well depths, known as the twin tub process.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/well_creation2.png)

4. Formation of gate
  - The gate is a pivotal CMOS transistor terminal that controls threshold voltages for transistor switching.
  - A polysilicon layer is deposited and photolithography techniques are applied to create NMOS and PMOS gates.
  - Important parameters for gate formation include oxide capacitance and doping concentration.
  - Here gate voltage is controlled by doping, oxide capacitances and also made low resistance gate by additional doping of polysilicon with n-type impurities like Phosphorus and Arsenic.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/gate_formation.png)  

5. Lightly doped Drain(LDD) formation
  -  LDD regions are intentionally created in the transistor structure to mitigate problems like hot electron injection and short-channel effects.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/LDD.png)

  - As shown in the figure, LDD regions are kept intact by side-wall spacers which are formed by a process called plasma anisotropic etching.
    
6. Source and Drain formation
  - A thin layer of screen oxide is added to avoid channeling during implants
  - N+ and P+ implants are formed by a process called High-temperature Annealing which involves subjecting silicon wafers or substrates to carefully controlled high-temperature environments for specified durations.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/source_drain_formation.png)

7. Contacts and Interconnects formation
  - Previously deposited thin oxide layer is etched off using Hydrofluoric solution
  - Deposit Titanium for low resistant contacts on wafer using sputtering
  - Wafer is heated in Nitrogen at 700-900 degrees which results in the formation of low-resistant titanium silicon dioxide for interconnect contacts and titanium nitride for top-level connections, enabling local communication.
  - Extra Tin is etched using a process called RCA cleaning.
    
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/interconnects.png)  

8. Higher level metal formation
  - To bring our metal contacts, non-planar topography is not suitable. Hence, a thick layer of SiO2 doped with phosphorus or Boron is deposited on the surface of the wafer and wafer surface is planarized using a process called Chemical mechanical Polishing.
  - TiN and blanket Tungsten layers are deposited and subjected to CMP.
  - An aluminum (Al) layer is added and subjected to photolithography and CMP.
  - This constitutes the first level of interconnects, and additional interconnect layers are added to reach higher-level metal layers.
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/metal_contacts.png)

The 16 masks used in the above process are:  

- *Substrate Mask (Mask 1):* This mask defines the active regions on the silicon wafer where transistors and other devices will be formed. It specifies the boundaries of the N-well and P-well regions.
- *Threshold Voltage Adjustment Mask (Mask 2):* This mask adjusts the threshold voltage of the transistors by defining the regions where threshold voltage implants are required.
- *Gate Oxide Mask (Mask 3):* This mask defines the areas where gate oxide will be grown or deposited. The gate oxide acts as an insulator between the gate electrode and the silicon substrate.
- *Poly-Silicon Gate Mask (Mask 4):* This mask defines the gate electrodes for both N-channel and P-channel transistors. It outlines the shape of the gates.
- *N+ and P+ Diffusion Masks (Masks 5 and 6):* These masks define the source and drain regions for the N-channel and P-channel transistors, respectively. These regions are typically doped with impurities to create the necessary electrical characteristics.
- *Contact Mask (Mask 7):* This mask defines the openings for contacts, which allow the metal layers to connect to the underlying silicon.
- *First Metal Layer Mask (Mask 8):* This mask defines the first layer of metal interconnects that connect various components on the chip, such as transistors and contacts.
- *Interlayer Dielectric (ILD) Mask (Mask 9):* This mask defines the dielectric material that insulates metal layers from each other. It also specifies the locations of vias for vertical connections.
- *Via Mask (Mask 10):* This mask defines the openings in the ILD layer for vias, which enable vertical connections between metal layers.
- *Second Metal Layer Mask (Mask 11):* This mask defines the second layer of metal interconnects, which connect to the underlying metal layer and vias.
- *Barrier Layer Mask (Mask 12):* This mask defines layers used to improve adhesion between metal and dielectric, enhancing the reliability of the interconnects.
- *Third Metal Layer Mask (Mask 13):* This mask defines the third layer of metal interconnects, which can connect to the lower metal layers through vias.
- *Passivation Layer Mask (Mask 14):* This mask defines the protective passivation layer that covers the entire chip, protecting it from external factors and contamination.
- *Bond Pad Mask (Mask 15):* This mask defines the locations of bond pads, which are used for external electrical connections and testing.
- *Test Structure Mask (Mask 16):* This mask includes various test structures used for quality control, testing, and characterization during manufacturing.


</details>  
<details>
  <summary>SKY130 Tech File</summary>  
 <br>   
  
**Magic Tool options and DRC Rules**  

The technology file is a setup file that declares layer types, colors, patterns, electrical connectivity, DRC, device extraction rules and rules to read LEF and DEF files.
Magic layouts can be sourced from [opencircuitdesign.com](https://opencircuitdesign.com/) using the command:
```
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
tar xfz drc_tests.tgz
```
![drc_tests folder contents](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/drc_test_contents_.png)

The ```.magicrc``` loads locally the tech file required by the user. Since this file sets up the tech file, sky130.tech need not be mentioned in the command used to invoke Magic. Hence Magic can be invoked more conveniently:
```
magic -d XR
```
**DRC Errors**  

To analyse DRC errors, magic is invoked and the met3.mag file is opened either from the software as ```file-> open-> met3.mag``` or by running command in tkcon as ```magic -d XR met3```.   
DRC errors can be found by selecting a component and typing: ```drc why``` in tkcon.

![drc error checking in magic](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/drc_error.png)  

met3.5 is the name of a periphery rule. The descriptions of DRC rules can be found in the [SKY130 PDK’s documentation](https://skywater-pdk.readthedocs.io/en/main/rules/)

To check for vias in the metal3 layer, make a rectangluar selection in an empty space and paint it with the m3contact color from the color palette by clicking middle mouse button or by typing the below command in tkcon window:  
```paint m3contact```

The metal cuts in vias can be viewed by: ```cif see VIA2```

![vias](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/m3contact.png)  

In this fashion, one can search for DRC errors, read up their descriptions and resolve them by editing the technology file.


</details>  

## Day 4 - Timing Analysis using OpenSTA  
<details>
  <summary>Timing modeling using delay tables</summary>  

Alignment of the input and output ports is specified in the ```tracks.info``` file. Follow the below command to view the file :  
```
cd /home/rachana/open_pdks/sky130/openlane/sky130_fd_sc_hd
gedit tracks.info
```
Accessing the tracks.info file for the pitch and direction information:

```
li1 X 0.23 0.46
li1 Y 0.17 0.34
met1 X 0.17 0.34
met1 Y 0.17 0.34
met2 X 0.23 0.46
met2 Y 0.23 0.46
met3 X 0.34 0.68
met3 Y 0.34 0.68
met4 X 0.46 0.92
met4 Y 0.46 0.92
met5 X 1.70 3.40
met5 Y 1.70 3.40
```
The CMOS Inverter ports A and Y are on li1 layer. It needs to be ensured that they're on the intersection of horizontal and vertical tracks. 
To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. Convergence of grid and tracks can be achieved using the following command:
```
grid 0.46um 0.34um 0.23um 0.17um
```
After typing the command, in the Magic window it is seen that ports A and Y are present on the intersection point of grid lines.  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/tracks_info.png)  

**Standard Cell LEF generation**  

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. The easiest way to define a port is through Magic Layout window and following are the steps:  

In Magic Layout window, first source the .mag file for the design (here inverter). Then Edit >> Text which opens up a dialogue box.  

![image](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/assets/140998470/26041303-5c8a-44d4-b6a3-8c94f24a3ff6)  

For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure.  
Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port class signal
```
Select VPWR area
```
port class inout
port use power
```

Select VGND area
```
port class inout
port use ground
```

LEF extraction can be carried out in tkcon as follows:
```
lef write
```
This generates ```sky130_vsdinv.lef``` file.

**Integrating custom cell in OpenLANE**

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.tcl``` must be modified:
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a -tag RUN_2023.09.10_12.07.03 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/synthesis.png)  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/sta_log.png)  

Next floorplan is run, followed by placement:  
```
run_floorplan
run_placement
```
To check the layout invoke magic from the ```results/placement``` directory:

```
magic -T /home/rachana/OpenLane/vsdstdcelldesign/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &
```

**Delay Tables**  

Basically, Delay is a parameter that has huge impact on our cells in the design. Delay decides each and every other factor in timing. For a cell with different size, threshold voltages, delay model table is created where we can it as timing table. Delay of a cell depends on input transition and out load. Lets say two scenarios, we have long wire and the cell(X1) is sitting at the end of the wire : the delay of this cell will be different because of the bad transition that caused due to the resistance and capcitances on the long wire. we have the same cell sitting at the end of the short wire: the delay of this will be different since the tarn is not that bad comapred to the earlier scenario. Eventhough both are same cells, depending upon the input tran, the delay got chaned. Same goes with o/p load also.

VLSI engineers have identified specific constraints when inserting buffers to preserve signal integrity. They've noticed that each buffer level must maintain consistent sizing, but their delays can vary depending on the load they drive. To address this, they introduced the concept of "delay tables," which essentially consist of 2D arrays containing values for input slew and load capacitance, each associated with different buffer sizes. These tables serve as timing models for the design.

When the algorithm works with these delay tables, it utilizes the provided input slew and load capacitance values to compute the corresponding delay values for the buffers. In cases where the precise delay data is not readily available, the algorithm employs a technique of interpolation to determine the closest available data points and extrapolates from them to estimate the required delay values.  
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/assets/140998470/d9ffb286-711a-4664-b6ba-db27a8be1c45)  

**Introduction to Clock Jitter and Uncertainity**  

*Clock Jitter:*

Clock jitter refers to the variation or deviation in the timing of a clock signal from its ideal or expected timing. Clock signal is used to synchronize various operations and components. Ideally, the clock signal should have a constant and precise period, but in reality, due to various factors, the timing can fluctuate. These fluctuations can be in the form of random variations (random jitter) or systematic variations (deterministic jitter).  
  
*Uncertainty:*

Uncertainty in the context of clocks and timing refers to the lack of precise knowledge about the exact timing or synchronization. This uncertainty can arise from various sources, including but not limited to:

- Measurement inaccuracies or limitations in the measurement instruments.
- Variability in the clock signal due to environmental factors or component variations.
- Limitations in the accuracy of the clock generation circuitry.

![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/clocl_jitter.png)  
</details>
<details>  
  <summary>Post Synthesis Timing Analysis</summary>  
<br>  

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, a new file pre_sta.conf is created. This file would be reqiured to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:  
```
sta pre_sta.conf
```
Since clock tree synthesis has not been performed yet, the analysis is with respect to ideal clocks and only setup time slack is taken into consideration. The slcak value is the difference between data required time and data arrival time. The worst slack value must be greater than or equal to zero. If a negative slack is obtained, following steps may be followed:
1. Change synthesis strategy, synthesis buffering and synthesis sizing values 
2. Review maximum fanout of cells and by upsizing the cells i.e replace cells with high drive strength
```
set ::env(MAX_FANOUT_CONSTRAINT) 4
4
```
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/replace_cell.png)


</details>  
<details>
  <summary>Clock Tree Synthesis</summary>  
  <br>  
The purpose of building a clock tree is enable the clock input to reach every element and to ensure a zero clock skew. H-tree is a common methodology followed in CTS. Before attempting a CTS run in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the write_verilog command. Then, the synthesis, floorplan and placement is run again.  
  
Here's a detailed breakdown of the process and its significance:  

Objective of CTS:  

The primary objective of CTS is to distribute the clock signal generated by the clock source (oscillator or PLL) to all the sequential elements (e.g., flip-flops) in the design in a way that:  
- Minimizes clock skew: Clock skew is the variation in arrival times of the clock signal at different points in the circuit. Minimizing this variation is crucial for proper synchronization.  

- Balances the load: The clock distribution network should evenly distribute the load to avoid signal integrity issues and ensure consistent performance across the circuit.
Process of CTS:  

A. Clock Tree Construction:  

Starting with the clock source, the clock tree synthesis algorithm constructs a tree-like structure, branching out to different regions of the design.
The structure consists of buffers, inverters, and other elements that help distribute the clock signal effectively.  

B. Buffer Insertion:  

- Buffers are inserted strategically in the clock tree to balance the load and minimize clock skew.  

- The placement and sizing of these buffers are optimized to meet timing requirements while considering power consumption and area constraints.  

C. Clock Skew Optimization:  

Clock skew optimization involves adjusting the placement and sizing of buffers to minimize clock skew and achieve a balanced clock distribution.
Techniques like buffer resizing, buffer replication, and buffer reordering may be used to achieve optimal skew.  

**Cross Talk in VLSI**  
Crosstalk in Very Large Scale Integration (VLSI) refers to unwanted electromagnetic or capacitive/inductive coupling between adjacent conductors, such as wires or traces, on an integrated circuit. This phenomenon can lead to signal interference and degradation in the quality of transmitted signals. Hence, shielding is technique used to protect critical nets from crosstalk and clock net is one of the critical nets where shielding has to be implemented.  

The purpose of building a clock tree is enable the clock input to reach every element and to ensure a zero clock skew. H-tree is a common methodology followed in CTS.
Before attempting a CTS run in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. To run CTS use the below command:

```
run_cts
```

The CTS run adds clock buffers in therefore buffer delays come into picture and our analysis from here on deals with real clocks. Setup and hold time slacks may now be analysed in the post-CTS STA anlysis in OpenROAD within the openLANE flow:  

```
openroad
read_lef /home/rachana/OpenLane/designs/picorv32a/runs/RUN_2023.09.16_11.04.25/tmp/merged.max.lef
read_def /home/rachana/OpenLane/designs/picorv32a/runs/RUN_2023.09.16_11.04.25/results/cts/picorv32a.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /home/parallels/OpenLane/designs/picorv32a/runs/RUN_09-09_11-20/results/synthesis/picorv32a.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
read_sdc /home/parallels/OpenLane/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```





</details>

## References
1. https://openlane.readthedocs.io/en/latest/
2. https://woset-workshop.github.io/PDFs/2020/a21.pdf
3. https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130#components-of-opensource-digital-asic-design
4. https://github.com/kanishr1/
5. https://github.com/aasthadave9/Advanced-Physical-Design-Using-OpenLANE-Sky130/
6. https://github.com/nickson-jose/vsdstdcelldesign/
7. http://opencircuitdesign.com/magic/
8. https://skywater-pdk.readthedocs.io/en/main/
9. 



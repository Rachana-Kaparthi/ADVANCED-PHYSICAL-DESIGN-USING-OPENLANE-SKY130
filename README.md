# ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130
[Day 1- Inception of open-source EDA,Openlane and Sky130 PDK](#day-1---inception-of-open-source-edaopenlane-and-sky130-pdk)  

[Day 2 - Good flooplan vs Bad Floorplan and introduction to library cells](#day-2---good-flooplan-vs-bad-floorplan-and-introduction-to-library-cells)  

[Day 3 - Design Library cell using Magic Layout and NG Spice Characterisation](#day-3---design-library-cell-using-magic-layout-and-ng-spice-characterisation)

[References](#references)


## Day 1 - Inception of open-source EDA,Openlane and Sky130 PDK
<details>
<summary>Installation of Required Tools</summary>  
  
**OpenLane**  
  
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

**OpenSTA:**  

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

**Magic**  
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

  - High-temperature furnace processes drive-in diffusion to establish well depths, known as the tub process.
![](https://github.com/Rachana-Kaparthi/ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130/blob/main/images/well_creation2.png)





</details>

## References
1. https://openlane.readthedocs.io/en/latest/
2. https://woset-workshop.github.io/PDFs/2020/a21.pdf
3. https://github.com/Devipriya1921/Physical_Design_Using_OpenLANE_Sky130#components-of-opensource-digital-asic-design
4. https://github.com/kanishr1/
5. https://github.com/aasthadave9/Advanced-Physical-Design-Using-OpenLANE-Sky130/



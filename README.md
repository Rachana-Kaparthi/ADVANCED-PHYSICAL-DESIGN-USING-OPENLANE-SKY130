# ADVANCED-PHYSICAL-DESIGN-USING-OPENLANE-SKY130
[Day 1- Inception of open-source EDA,Openlane and Sky130 PDK](#day-1---inception-of-open-source-edaopenlane-and-sky130-pdk)

[References](#references)


## Day 1 - Inception of open-source EDA,Openlane and Sky130 PDK
<details>
<summary>Installation of Openlane</summary>
  
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
```

**Installation of OpenSTA:**  

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

*Physical verifications:*  

  *Design Rule Checking (DRC):*

    Verify that the chip layout adheres to the manufacturing process's design rules. DRC tools like Magic or KLayout are commonly used for this purpose.
  *Layout vs. Schematic (LVS) Check:*

    Ensure that the final layout matches the original schematic. LVS tools like Netgen or Calibre are used to compare the netlist extracted from the layout with the synthesized netlist.  

*Timing verifications:*

  *Static Timing Analysis (STA):*

    Analyze the timing characteristics of your design to ensure that all setup and hold time requirements are met. OpenSTA is commonly used in the OpenLane flow for STA.

  
</details>

## References
1. https://openlane.readthedocs.io/en/latest/
2. 



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
  
</details>

## References
1. https://openlane.readthedocs.io/en/latest/
2. 



# Physical Design with OpenLane using SKY130 PDK
## Table of Contents

## Introduction
With the advent of open-source technologies for Chip development, there were several RTL designs, EDA Tools which were open-sourced. The missing piece in a complete Open source chip development was filled by the [SKY130 PDK](https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google.  There were several EDA Tools, which played specfic roles in the design cycle. There was not a clean design flow and Skywater pdk was compatible with only the industrty tools.  [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) addressed these issues in providing a completely automated and clean RTL to GDSII flow. OpenLane is not a tool, but a flow which consists of several EDA tools, automation scripts and Skywater-pdks tuned to work specifically with the open-source EDA tools.     

## Overall Design Flow
For a design Specification an RTL Design is written in HDLs like Verilog /VHDL or RTL Design is generated using Hardware Construction Languages like Chisel or High Level Synthesis using  SystemC, MATLAB HDL Coder, Bluespec etc or a modern abstraction level called [TL-Verilog](makerchip.com) (its not a HDL/HLS) , specified by TL-x.org.
After this begins the workflow of taking the RTL Netlist into a fabricated IC, which is called as Physical Design Flow.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement of Logical Synthesis. Now we do CTS (Clock Tree Synthesis) such there the skew of the clock is the minimum or within the required threshold. After CTS, Routing is done to route all the components placed. Between each and every step that happens in the physical design flow starting from Logic Synthesis to routing, a procedure called "Static Timing Analysis" is done to analyse the design at every step to ensure the actual correctness of the design.  To view every stage, Magic is an open source tool to view the layouts. A small netlist can be extracted and a SPICE Simulation can be performed and compared with the Post Layout Simulation using ngspice.

![Simplified Design Flow](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/1.PNG)

## OpenLane Flow

![enter image description here](https://github.com/The-OpenROAD-Project/OpenLane/blob/master/docs/_static/openlane.flow.1.png)

### 1.  Synthesis 
The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then  converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts  into  a  RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants 

### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

### 1.2 Deign Exploration Utility 
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design 

###  2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin. 

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source. 

### 3. Placement
There are two types of placement.  The other required logic is placed optimally.
Placement is of two steps
- Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
- Detialed Placement - After Global placement is done minimal alterations are done to correct the issues

### 4. Clock Tree Synthesis 
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges
- OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
- OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode
OpenLane allows the user to chose either of the above approaches

###  5. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

 - Global Routing - This is done inside the OpenROAD flow (FastRoute)
 - Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the **Logic Equivalence Check** is performed by Yosys, since OpenROAD does some optimisations the circuit.  

### 6. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

### 7. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.  

### 8. Sign-off Steps
- Design Rule Check (DRC) is performed by Magic
- Layout Versus Schematic (LVS) is performed by Netgen

### 9. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file 

## OpenLane Installation and Environment Setup
Refer
 - [Efabless GIthub](https://github.com/The-OpenROAD-Project/OpenLane)   or
 - [OpenLane build Script by Nikson Jose](https://github.com/nickson-jose/openlane_build_script)
 - The above repository can be followed if the installation is done on a VirtualMachine/Linux 
 - The following steps are required to  **run OpenLane inWindows Subsystem for Linux (WSL1)** before installation of 
 - OpenLANE Enable WSL1 - 
	 - [Reference](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
	 - Install VSCode and RemoteWSL Extension 
	 - Connect to the WSL and open  the folder in WSL 
	 - Download and Install  [VcXserv](https://sourceforge.net/projects/vcxsrv/) to run GUI Applications
	 -  Start VcXserv. Check the "Disable access control box" 
	 -   Set the Display number as 0 (or anynumber)  In WSL terminal use the  command`export DISPLAY=:0`  
	 -  Install [Docker Desktop](https://www.docker.com/products/docker-desktop) in windows 
	 -    Enable the below option 		
	 - ![Docker  Setup](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/docker1.png)
	 - Follow [this](https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly) and install docker dependencies inside WSL
	 -    Every time start docker   in WSL to use the docker in windows exposed on the port 2375 using  this command
     	   ```echo "export DOCKER_HOST=tcp://localhost:2375" >> ~/.bashrc && source ~/.bashrc```
		   This must be done everytime before trying to OpenLane 
		    Use ```docker info``` to check the status 
		    
    This installation can also be  done on a remote Linux instance and Putty can be used with X11 fowarding configured to ```localhost:0``` with VcXsrv installed in   the host machine with Display number set to 0.

## OpenLane Directory Structure
Open the openlane directory
![enter image description here](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/dirstr.png)

 - The ```designs``` folder contains all the designs provided by Efabless. This is the directory from which OpenLane fetches the design
 - The ```scripts``` folder contains all the automation scripts used by OpenLane
 -  Open in the ```pdk``` folder contains three sub folders. 
 - ```skywater-pdk``` is by defaukt not configured to work with opensource tools. So OpenLane provides ```open_pdk``` and ```Sky130A``` directory which has the configuration files for each of the tools used in the OpenLane flow
 
 ![pdk directory](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/sky130a.PNG)

### Design folder
 Consider the picorv32a design. Upon design preparation a runs folder is added. Within the folder containing the date resides the configuration, results, reports and other files that are use in the run. 
 
![Design](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/design2.PNG)

## Working with OpenLane
Go the the openlane directory and type ```docker``` to start the docker containter.
The terminal changes into the docker instance.
Open the OpenLane in interactive mode.
```./flow.tcl -interactive```
Set the package required by OpenLane.
```package require openlane 0.9```

![OpenLane 1](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK/blob/main/images/o1.png)

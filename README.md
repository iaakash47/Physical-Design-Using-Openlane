# Physical-Design-Using-Openlane

This repository contains all the information of Advanced Physical Design Using OpenLANE workshop. It is primarily foucused on a complete RTL2GDS flow using the open-soucre flow named OpenLane.picorv32a RISC-V core design is used for the purpose.
In this project a complete RTL to GDSII flow for  PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom desgined standard cells with Sky130 PDK are also used in the flow.  Timing Optimisations are carried out. Slack violations are removed. DRC is verified


## Table of Contents

  * [Introduction](#introduction)
  * [Overall Design Flow](#overall-design-flow)
  * [OpenLane Flow](#openlane-flow)
    + [1.  Synthesis](#1--synthesis)
    + [1.1 Synthesis Strategies](#11-synthesis-strategies)
    + [1.2 Deign Exploration Utility](#12-deign-exploration-utility)
    + [1.3 Design For Test - DFT Insertion](#13-design-for-test---dft-insertion)
    + [2. Floor Planning and Power Planning](#2-floor-planning-and-power-planning)
    + [3. Placement](#3-placement)
    + [4. Clock Tree Synthesis](#4-clock-tree-synthesis)
    + [5. Fake Antenna and diode swapping](#5-fake-antenna-and-diode-swapping)
    + [5. Routing](#5-routing)
    + [6. RC Extraction](#6-rc-extraction)
    + [7. STA](#7-sta)
    + [8. Sign-off Steps](#8-sign-off-steps)
    + [9. GDSII Extraction](#9-gdsii-extraction)
  * [OpenLane Installation and Environment Setup](#openlane-installation-and-environment-setup)
  * [OpenLane Directory Structure](#openlane-directory-structure)
  * [Working with OpenLane](#working-with-openlane)
    + [Start Openlane](#start-openlane)
    + [Design Preparation](#design-preparation)
    + [Configuration Priority](#configuration-priority)
  * [Synthesis](#synthesis)
    + [Key concepts](#key-concepts)
      - [Utilisation Factor](#utilisation-factor)
       + [Aspect Ratio](#aspect-ratio)
  * [Floorplanning](#floorplanning)
    + [Pre-Placed cells](#pre-placed-cells)
    + [Decoupling Capacitors to the pre placed cells](#decoupling-capacitors-to-the-pre-placed-cells)
    + [Power Planning](#power-planning)
    + [Pin Placement](#pin-placement)
    + [Floorplanning - Openlane](#floorplanning---openlane)
  * [Placement](#placement)
 * [Cell Design Flow](#cell-design-flow)
      - [SPICE Deck Creation](#spice-deck-creation)
      - [Simulation in ngspce](#simulation-in-ngspce)
      - [VTC](#vtc)
      - [VTC with 2.5 x W (2.5 times channel width of pmos](#vtc-with-25-x-w---5-times-channel-width-of-pmos-)
      - [Transient Simulation](#transient-simulation)
 * [Custom Design of SKY130 Standard cell](#custom-design-of-sky130-standard-cell)
      - [SPICE Characterisation](#spice-characterisation)
      - [LEF Extraction](#lef-extraction)
  * [Synthesis, Floorplanning with custom standard cell](#synthesis--floorplanning-with-custom-standard-cell)
  * [Static Timing Analysis](#static-timing-analysis)
  * [Floorplanning and Placement](#floorplanning-and-placement)
  *  [CTS](#cts)
  * [Pre-CTS Timing Analysis in OpenRoad](#pre-cts-timing-analysis-in-openroad)
* [PDN](#pdn)
* [Routing](#Routing)
* [GDSII](#gdsii)
* [Acknowledgements](#acknowledgements)
* [References](#references)

 All the steps are further discussed in details in the repository.
 
## Introduction
With the advent of open-source technologies for Chip development, there were several RTL designs, EDA Tools which were open-sourced. The missing piece in a complete Open source chip development was filled by the [SKY130 PDK](https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google.  There were several EDA Tools, which played specfic roles in the design cycle. There was not a clean design flow and Skywater pdk was compatible with only the industrty tools.  [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) addressed these issues in providing a completely automated and clean RTL to GDSII flow. OpenLane is not a tool, but a flow which consists of several EDA tools, automation scripts and Skywater-pdks tuned to work specifically with the open-source EDA tools.
The current release target to a SKY130 (i.e. 130 nm) process node is available as [SkyWater Open Source PDK](https://github.com/google/skywater-pdk). The PDK provides Physical VLSI Designer with a wide range of flexibility in design choices. All the designs and simulations listed in this repository are carried out using the same SkyWater Open Source PDK.

RTL to GDSII Flow refers to the all the steps involved in converting a logical Register Transfer Level(RTL) Design to a fabrication ready GDSII format. GDSII is a database file format which is an industry standard for data exchange of IC layout artwork.
  The RTL to GSDII flow consists of following steps:
  - RTL Synthesis
  - Static Timing Analysis(STA)
  - Design for Testability(DFT)
  - Floorplanning
  - Placement
  - Clock Tree Synthesis(CTS)
  - Routing
  - GDSII Streaming
 
 All the steps are further discussed in details in the repository.
 
 # About Google SkyWater PDK
 
Google and SkyWater Technology Foundry in collaboration have released a completely open-source Process Design Kit(PDK) in May, 2020. The current release target to a SKY130 (i.e. 130 nm) process node is available as [SkyWater Open Source PDK](https://github.com/google/skywater-pdk). The PDK provides Physical VLSI Designer with a wide range of flexibility in design choices. All the designs and simulations listed in this repository are carried out using the same SkyWater Open Source PDK.


## Overall Design Flow
For a design Specification an RTL Design is written in HDLs like Verilog /VHDL or RTL Design is generated using Hardware Construction Languages like Chisel or High Level Synthesis using  SystemC, MATLAB HDL Coder, Bluespec etc

  | Name of Tool | Application / Usage |
  | --- | --- |
  | [Yosys](https://github.com/YosysHQ/yosys) | Synthesis of RTL Design |
  | ABC | Mapping of Netlist |
  | [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA) | Static Timing Analysis |
  | [OpenROAD](https://github.com/The-OpenROAD-Project/OpenROAD) | Floorplanning, Placement, CTS, Optimization, Routing |
  | [TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute) | Detailed Routing |
  | [Magic VLSI](http://opencircuitdesign.com/magic/) | Layout Tool |
  | [NGSPICE](https://github.com/imr/ngspice) | SPICE Extraction and Simulation |
  | SPEF_EXTRACTOR | Generation of SPEF file from DEF file |
  
  After this begins the workflow of taking the RTL Netlist into a fabricated IC, which is called as Physical Design Flow.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement of Logical Synthesis. Now we do CTS (Clock Tree Synthesis) such there the skew of the clock is the minimum or within the required threshold. After CTS, Routing is done to route all the components placed. Between each and every step that happens in the physical design flow starting from Logic Synthesis to routing, a procedure called "Static Timing Analysis" is done to analyse the design at every step to ensure the actual correctness of the design.  To view every stage, Magic is an open source tool to view the layouts. A small netlist can be extracted and a SPICE Simulation can be performed and compared with the Post Layout Simulation using ngspice.

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
The above list of tools shows that, many different tools are required for various tasks in Physical VLSI Design. Each tool in itself have number of system requirements and require various supporting tools to be installed. Installing each tool one-by-one seems in-efficient. This is made easy by some custom scripts that setup the required tools and environment for them in just a few easy steps. To install all the required tools, one can refer to the below mentioned repositories:
  - [VSDFlow](https://github.com/kunalg123/vsdflow) - Installs Yosys, Magic, OpenTimer, OpenSTA and some other supporting tools
  - [OpenLANE Build Scripts](https://github.com/nickson-jose/openlane_build_script) - Install all required OpenROAD and some supporting tools

## OpenLane Directory Structure
Open the openlane directory
![Screenshot from 2022-08-20 18-32-20](https://user-images.githubusercontent.com/88897605/185748211-64e57fc9-0438-4776-bcbb-e8e89feb7d53.png)

- The ```designs``` folder contains all the designs provided by Efabless. This is the directory from which OpenLane fetches the design.  Consider the picorv32a design. Upon design preparation a runs folder is added. Within the folder containing the date resides the configuration, results, reports and other files that are use in the run. 
  
![Design](./images/design2.PNG)
 - The ```scripts``` folder contains all the automation scripts used by OpenLane
 -  Open in the ```pdk``` folder contains three sub folders. 
 - ```skywater-pdk``` is by defaukt not configured to work with opensource tools. So OpenLane provides ```open_pdk``` and ```Sky130A``` directory which has the configuration files for each of the tools used in the OpenLane flow
 - The `configuration` folder comtains the .tvl configurations for each tool. However these configurations can be overridden within the design or interactively in the openlane flow
- The `pdk` directory contains
    - `skywater-pdks` - This contains the pdk provided by the skywater foundry
    - `openpdk` - This contains the openpdks
    - `sky130A` - This directory contains the library referenes and the library technology files which are adpated to work with OpenLane Flow

## Working with OpenLane

### Start Openlane

Go the the openlane directory and type ```docker``` to start the docker containter.\

The terminal changes into the docker instance.\

Open the OpenLane in interactive mode.\

```./flow.tcl -interactive```\

Set the package required by OpenLane.\

```package require openlane 0.9```
### Design Preparation 

Prepare the design

```prep -design picorv32a```

- To resume from a previous run use `-tag run_name`
- To overwrite the previous run use `tag run_name -overwrite`
- *Note*: Any configuration done in the `config.tcl` of the source folder after design preparation will not be refleceted. To run wih a modified configuration, the design configuration can be overriten by passing the configuration to openlane interactively
- A runs folder is created as discussed
- On loading a previous run, to know the last run state one has to check the Current def file which is set. This can be done using
```zsh
echo $::env(CURRENT_DEF)
```
- To set to resume from a stage before the current DEF , one has to set the ```CURRENT_DEF``` environment variable to the required path. This can be done using
```tcl
set ::env(CURRENT_DEF) /path/to/the/required/def/file
```
- The `def` files of every  stage can be found in the `runs>results>stage_name>design_stage.def` path. 
- These `def` files can be opened with `magic` by using the `sky130A.tech` as the technology file and the `lef` file from the `tmp` directory if required.

### Configuration Priority

Configuration priority (from high to low) is as follows
- `pdk_specific_config.tcl`  - Design Folder
- `config.tcl` - Design Folder
- `tool_specific_config` - Configuration Folder in OPENLANE_ROOT

## Synthesis

Run the synthesis

```run_synthesis```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports 



View the synthesis statistics
```


24. Printing statistics.

=== picorv32 ===

   Number of wires:               9379
   Number of wire bits:           9761
   Number of public wires:        1512
   Number of public wire bits:    1894
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               9659
     sky130_fd_sc_hd__a2111o_2       1
     sky130_fd_sc_hd__a211o_2       68
     sky130_fd_sc_hd__a211oi_2      11
     sky130_fd_sc_hd__a21bo_2       17
     sky130_fd_sc_hd__a21boi_2       6
     sky130_fd_sc_hd__a21o_2       263
     sky130_fd_sc_hd__a21oi_2      117
     sky130_fd_sc_hd__a221o_2      119
     sky130_fd_sc_hd__a22o_2       155
     sky130_fd_sc_hd__a22oi_2        2
     sky130_fd_sc_hd__a2bb2o_2      22
     sky130_fd_sc_hd__a311o_2       35
     sky130_fd_sc_hd__a311oi_2       1
     sky130_fd_sc_hd__a31o_2        80
     sky130_fd_sc_hd__a31oi_2        7
     sky130_fd_sc_hd__a32o_2       108
     sky130_fd_sc_hd__a41o_2         3
     sky130_fd_sc_hd__and2_2       218
     sky130_fd_sc_hd__and2b_2       29
     sky130_fd_sc_hd__and3_2       110
     sky130_fd_sc_hd__and3b_2       41
     sky130_fd_sc_hd__and4_2        44
     sky130_fd_sc_hd__and4b_2        1
     sky130_fd_sc_hd__buf_1       2613
     sky130_fd_sc_hd__buf_2         18
     sky130_fd_sc_hd__conb_1       106
     sky130_fd_sc_hd__dfxtp_2     1596
     sky130_fd_sc_hd__inv_2         69
     sky130_fd_sc_hd__mux2_1         1
     sky130_fd_sc_hd__mux2_2      1629
     sky130_fd_sc_hd__mux4_2       440
     sky130_fd_sc_hd__nand2_2      229
     sky130_fd_sc_hd__nand2b_2       1
     sky130_fd_sc_hd__nand3_2       13
     sky130_fd_sc_hd__nand3b_2       4
     sky130_fd_sc_hd__nand4_2        2
     sky130_fd_sc_hd__nor2_2       226
     sky130_fd_sc_hd__nor2b_2        1
     sky130_fd_sc_hd__nor3_2        13
     sky130_fd_sc_hd__nor3b_2        3
     sky130_fd_sc_hd__nor4_2         4
     sky130_fd_sc_hd__nor4b_2        2
     sky130_fd_sc_hd__o2111a_2       4
     sky130_fd_sc_hd__o2111ai_2      4
     sky130_fd_sc_hd__o211a_2       94
     sky130_fd_sc_hd__o211ai_2       5
     sky130_fd_sc_hd__o21a_2       203
     sky130_fd_sc_hd__o21ai_2      118
     sky130_fd_sc_hd__o21ba_2        9
     sky130_fd_sc_hd__o21bai_2       4
     sky130_fd_sc_hd__o221a_2       67
     sky130_fd_sc_hd__o22a_2        45
     sky130_fd_sc_hd__o2bb2a_2       6
     sky130_fd_sc_hd__o311a_2        5
     sky130_fd_sc_hd__o31a_2        15
     sky130_fd_sc_hd__o31ai_2        8
     sky130_fd_sc_hd__o32a_2         3
     sky130_fd_sc_hd__o32ai_2        1
     sky130_fd_sc_hd__o41a_2         2
     sky130_fd_sc_hd__or2_2        385
     sky130_fd_sc_hd__or2b_2        23
     sky130_fd_sc_hd__or3_2         49
     sky130_fd_sc_hd__or3b_2        19
     sky130_fd_sc_hd__or4_2         33
     sky130_fd_sc_hd__or4b_2         5
     sky130_fd_sc_hd__xnor2_2       86
     sky130_fd_sc_hd__xor2_2        38

   Chip area for module '\picorv32': 100880.502400

```
The STA Reports can be viewed from the Reports folder.

The openSTA tool generated the timing reports. It can be seen from below that 

### Key concepts

#### Utilisation Factor 
```
Utilisation Factor =  Area occupied by netlist
                     __________________________
                        Total area of core
```


- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1613/14876 = 0.1084** (i.e: 10.8%) [From the synthesis statistics]
- 
A Utilisation Factor of 1 signifies 100% utilisation leaving no space for extra cells such as buffer. However, practically, the Utilisation Factor is 0.5-0.6. Likewise, an Aspect ratio of 1 implies that the chip is square shaped. Any value other than 1 implies rectanglular chip.

### Aspect Ratio

```
Aspect Ratio =  Height
               ________
                Width
```
- Aspect ratio is the ratio of height to the width of the die.
- Aspect Ratio of 1 indicates that the die is a square die

## Floorplanning

Floorplanning involves the following stages

### Pre-Placed cells

- Whenever there is a complex logic which is repeated multiple times or a design given by a third-party it can be perceived as abstract black box with input and output ports, clocks etc ., 
- These modules can be either macros or IP
    - Macro  - It is a module such as CPU Core which are developed by the entity fabicating the chip
    - IP - It is an "Intellectual Propertly" which the entity fabricating the chip gets as a package from a third party or even packaged Hard IPs developed by the same entity. Common examples of IPs are SRAM, PLL, Protocol Converters etc.,

- These Macros and IPs are placed in the core at first before placing the standard cells  and power planning
- These are optimally such that the cells which are more connected to each other are placed nearby and oriented for input and ouputs

### Decoupling Capacitors to the pre placed cells
- The power lines can have some RLC component causing the voltage to drop at the node where it enters the Blocks or the ground of the cell can be at a higher potential than ideally 0V
- When this happens, there is a chance such that the logic transitions are not to the upper or lower noise margins but to the forbidden state causing the circuit to misbehave
- This is prevented by adding a capacitor in parallel with the power and ground node of the block such that the capacitor decouples the block from the power source whenever there is a logic transition

### Power Planning

- When there are several cells or blocks drawing power from the same power rail and sinking power to the same ground pin the following effects are observed
    - Whenever there is alogic transition from 1 to 0 in a large number of cells then there is a Voltage Droop in the power lines as Voltage Drops from Vdd
    - Whener there is a logic transition from 0 to 1 in a large number of cells simultaneously causes the ground potential to raise above 0V calles as Ground Bump
    - These effects pose a risk of driving the logic state out of the specified noise margin.
    - To avoid this the Vdd and Gnd are placed as a grid of horizontal and vertical tracks and the cell nearer to an intersection can tap power or sink power to the Vdd or Gnd intersection respectively

### Pin Placement
 - The input, output and Clock pins are placed optimally such that there is less complication in routing or optimised delay
 - There are different styles of pin placement in openlane like `random pin placement` , `uniformly spaced` etc.,

### Floorplanning - Openlane

Command: `run_floorplan`

Successful floorplanning gives a `def` file as output. This file contains the die area and placement of standard cells.

![floorplan](https://user-images.githubusercontent.com/88897605/185786882-95ac2b3b-7545-4721-af5d-4c8fe27e821d.jpeg)

* Floorplan in Expanded View 

![d2_floorplan_magic_expand](https://user-images.githubusercontent.com/88897605/185786905-a463703b-3415-431c-a12a-5d2dde4c2614.jpeg)

* Floorplan in Klayout 

![Screenshot from 2022-08-21 15-45-00](https://user-images.githubusercontent.com/88897605/185786300-26756d66-642a-4994-b157-c390922cb02c.png)


he next step after floorplanning is placement. Placement determines location of each of the components on the die. Placement does not just place the standard cells available in the synthesized netlist. It also optimizes the design, thereby removing any timing violations created due to the relative placement on die.
   
## Placement 
- In this steps the standard cells are placed in the floorplanned design
- In palcement buffers are placed whereever the wire delay is large
- Placement in openlane happens in two steps
    - Global Placament
    - Detailed Placement
- Global placement is not always legalised
- However, Detailed placement is strict and adheres to the Design Rules
   Placement in OpenLANE is done using the following command. 
    
 ```run_placement```
   
The DEF file created during floorplan is used as an input to placement. Placement in OpenLANE occurs in two stages:
   - Global Placement
   - Detailed Placement
   
Placement is carried out as an iterative process till the value of overflow converges to 0.
   
Successful placement gives a `def` file as output.

![d2_placement_magic](https://user-images.githubusercontent.com/88897605/185787060-d69b4472-5297-4a98-95ce-bd7e0d24ce4f.jpeg)

* Placement in KLayout View 
![Screenshot from 2022-08-21 15-49-59](https://user-images.githubusercontent.com/88897605/185787105-2561d28c-d22d-4bca-8d13-2183417ef7e1.png)

 
The snippet below shows a layout for CMOS Inverter with and without design rule violations.
  
![Screenshot from 2022-08-21 14-38-06](https://user-images.githubusercontent.com/88897605/185787840-d52b173d-f8ec-47b4-8217-f630ada843d3.png)


![extract_spice](https://user-images.githubusercontent.com/88897605/185789526-eba7151f-8762-428d-bc01-36c1047e882e.jpeg)

* Waveform of inverter
![inverter_ngspice](https://user-images.githubusercontent.com/88897605/185789528-bd2ad88f-fc5c-4fca-9184-a3f908a51e48.jpeg)

## Magic Layout to Standard Cell LEF
  Before creating the LEF file we require some details about the layers in the designs. This details are available in a `tracks.info` as shown below. It gives information about the `offset` and `pitch` of a track in a given layer both in horizontal and vertical direction. The track information is given in below mentioned format.

![d4_track_info](https://user-images.githubusercontent.com/88897605/185789633-3f74076d-3e19-424f-ac70-427471953b94.jpeg)

  To create a standard cell LEF from an existing layout, some important aspects need to be taken into consideration.
  1. The height of cell be appropriate, so that the `VPWR` and `VGND` properly fall on the power distribution network.
  2. The width of cell should be an odd multiple of the minimum permissible grid size.
  3. The input and ouptut of the cell fall on intersection of the vertical and horizontal grid line.

### Standard Cell LEF generation
Before the CMOS Inverter standard cell LEF is extracted, the purpose of ports must be defined:
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


### Integrating custom cell in OpenLANE

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.tcl``` must be modified:
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a -tag 02-07_07-56 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```

Next floorplan is run, followed by placement:

```
init_floorplan
run_placement
```

To check the layout invoke magic from the ```results/placement``` directory:

Since the custom standard cell has been plugged into the openLANE flow, it would be visible in the layout:
![invadded](https://user-images.githubusercontent.com/88897605/185789824-fee07aa9-71e9-430b-b0e6-a6d286d941f4.png)

 ## Timing Analysis using OpenSTA
  The Static Timing Analysis(STA) of the design is carried out using the OpenSTA tool. The analysis can be done in to different ways.
  - Inside OpenLANE flow: This is by invoking `openroad` command inside the OpenLANE flow. In the openroad OpenSTA is invoked.
  - Outside OpenLANE flow: This is done by directly invoking OpenSTA in the command line. This requires extra configuration to be done to specific the verilog file, constraints, clcok period and other required parameters.
   
  OpenSTA is invoked using the below mentioned command.
  
    sta <conf-file-if-required>
  
  The above command gives an Timing Analysis Report which contains:
   1. Hold Time Slack
   2. Setup Time Slack
   3. Total Negative Slack (= 0.00, if no negative slack)
   4. Worst Negative Slack (= 0.00, if no negative slack)

```

Startpoint: _16415_ (rising edge-triggered flip-flop clocked by clk)
Endpoint: _16415_ (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                  0.15    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                  0.15    0.00    0.00 ^ _16415_/CLK (sky130_fd_sc_hd__dfxtp_2)
                  0.10    0.38    0.38 ^ _16415_/Q (sky130_fd_sc_hd__dfxtp_2)
     6    0.02                           count_cycle[55] (net)
                  0.10    0.00    0.38 ^ _12939_/A1 (sky130_fd_sc_hd__a21oi_2)
                  0.03    0.05    0.43 v _12939_/Y (sky130_fd_sc_hd__a21oi_2)
     1    0.00                           _00636_ (net)
                  0.03    0.00    0.44 v _16415_/D (sky130_fd_sc_hd__dfxtp_2)
                                  0.44   data arrival time

                  0.15    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                          0.25    0.25   clock uncertainty
                          0.00    0.25   clock reconvergence pessimism
                                  0.25 ^ _16415_/CLK (sky130_fd_sc_hd__dfxtp_2)
                         -0.02    0.23   library hold time
                                  0.23   data required time
-----------------------------------------------------------------------------
                                  0.23   data required time
                                 -0.44   data arrival time
-----------------------------------------------------------------------------
                                  0.20   slack (MET)
```

  
  If the design produces any setup timing violaions in the analysis, it can be eliminated or reduced using techniques as follows:
  1. Increase the clock period (Not always possible as generally operating frequency is freezed in the specifications)
  2. Scaling the buffers (Causes increase in design area)
  3. Restricting the maximum fan-out of an element. 

 ## Clock Tree Synthesis using TritonCTS
  Clock Tree Synthesis(CTS) is a process which makes sure that the clock gets distributed evenly to all sequential elements in a design. The goal of CTS is to minimize the clock latency and skew.
  There are several CTS techniques like:
  1. H - Tree
  2. X - Tree
  3. Fish bone
  
  In OpenLANE, clock tree synthesis is carried out using TritonCTS tool. CTS should always be done after the floorplanning and placement as the CTS is carried out on a `placement.def` file that is created during placement stage.
  
  The command used for running CTS in OpenLANE is given below.
  
    ```run_cts```
    
 This CTS is performed on the placement .def file. Since that is the recently run 

- In openlane type `openroad`
- Read the lef file
    - `read_lef /openLANE_flow/designs/picorv32a/runs/t3/tmp/merged.lef`
- Read the Def file
    - `read_def /openLANE_flow/designs/picorv32a/runs/t3/results/cts/picorv32a.placement.def`
- Create the db
    - `write_db pico_cts_2.db`
- Preform Analysis using OpenSTA inside openroad
- `read_db pico_cts_2.db`
- `read_verilog /openLANE_flow/designs/picorv32a/runs/t3/results/synthesis/picorv32a.v`
- `read_liberty $::env(LIB_SYNTH_COMPLETE)`
- `link_design picorv32a`
- `read_sdc /openLANE_flow/vsdstdcelldesign/extras/my_base.sdc`
- Set the clock buffer to use from 2
- `set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]`
- `set_propagated_clock [all_clocks]`
- `report_checks -format full_clock_expanded -digits 4`

![Screenshot from 2022-08-21 15-49-59](https://user-images.githubusercontent.com/88897605/185790437-ac0911ac-0a54-449e-b732-ea77b09739fe.png)


## Generation of Power Distribution Network
   In a normal RTL to GDSII flow the generation of power distribution network is done before the placement step, but in the OpenLANE flow generation of PDN is carried out after the Clock Tree Synthesis(CTS). This step generates all the tracks, rails required for routing power to entire chip.
   Generation of power distribution network is done using following command.
   
    ```gen_pdn```
    
- `gen_pdn` - Generate the Power Distribution network
- The power distrubution network has to take the `design_cts.def` as the input def file.
- This will create the grid and the straps for the Vdd and the ground. These are placed around the standard cells.
- The standard cells are designed such that it's height is multiples of the space between the Vdd and the ground rails. Here, the pitch is `2.72`. Only if the above conditions are adhered it is possible to power the standard cells.
- The power to the chip, enters through the `power pads`. There is each for Vdd and Gnd
- From the pads, the power enters the `rings`, through the `via`
- The `straps` are connected to the ring. Vdd straps are connected to the Vdd ring and the Gnd Straps are connected to the Gnd ring. There are horizontal and the vertical straps
- Now the power has to be supplied from the straps to the standard cells. The straps are connected to the `rails` of the standard cells
- If macros are present then the straps attach to the `rings` of the macros via the `macro pads` and the pdn for the macro is pre-done.
- There are definitions for the straps and the railss. In this design straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.

## Routing using TritonRoute
   OpenLANE uses TritonRoute, an open source router for modern industrial designs. The router consists of several main building blocks, including pin access analysis, track assignment, initial detailed routing, search and repair, and a DRC engine.
   The routing process is implemented in two stages:
   1. Global Routing - Routing guides are generated for interconnects
   2. Detailed Routing - Tracks are generated interatively.
   TritonRoute 14 ensures there are no DRC violations after routing.
   
   The following command is used for routing.
   
    ```run_routing```
    
- `run_routing` - To start the routing
- The options for routing can be set in the `config.tcl` file. 
- The optimisations in routing can also be done by specifying the routing strategy to use different version of `TritonRoute Engine`. There is a trade0ff between the optimised route and the runtime for routing.
- For the default setting picorv32a takes approximately 30 minutesaccording to the current version of TritonRoute.
- This routing stage must have the `CURRENT_DEF` set to `pdn.def`
- The two stages of routing are performed by the following engines
    - Global Route : Fast Route
    - Detailed Route : Triton Route
- Fast Route generates the routing guides, whereas Triton Route uses the Global Route and then completes the routing with some strategies and optimisations for finding the best possible path connect the pins.

![Screenshot from 2022-08-21 15-50-59](https://user-images.githubusercontent.com/88897605/185790608-d720606f-a4fb-4985-bf89-221506bf8b1b.png)

## GDSII

GDS Stands for Graphic Design Standard. This is the file that is sent to the foundry and is called "tape-out" 

*Fact- Earlier, the GDS files were written on magnetic tapes and sent out to the foundry and hence the name "tape-out"*

In openLane use the command `run_magic`

The GDSII file is generated in the `results/magic` directory

Checking DRC using `run_magic_drc`

![Screenshot from 2022-08-21 15-52-00](https://user-images.githubusercontent.com/88897605/185790695-3bffba4a-9a3c-4803-885b-4278c11a3050.png)

No DRC errors are found.

Opening the GDSII file in `klayout`

# Contributor
Aakash K</br>
Contact:iaakashkrish@gmail.com</br>

# Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd - kunalpghosh@gmail.com

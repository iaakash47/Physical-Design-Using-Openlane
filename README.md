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

10. Printing statistics.

=== picorv32a ===

   Number of wires:               6263
   Number of wire bits:           8542
   Number of public wires:         142
   Number of public wire bits:    1973
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               8115
     $_ANDNOT_                    1164
     $_AND_                        397
     $_DFFE_PP_                   1240
     $_DFF_P_                       91
     $_MUX_                       2709
     $_NAND_                       227
     $_NOR_                        168
     $_NOT_                        144
     $_ORNOT_                      164
     $_OR_                        1045
     $_SDFFCE_PN0P_                 34
     $_SDFFCE_PP0P_                  6
     $_SDFFE_PN0P_                 154
     $_SDFFE_PP0P_                   1
     $_SDFFE_PP1P_                   3
     $_SDFF_PN0_                    66
     $_SDFF_PP0_                     1
     $_XNOR_                       113
     $_XOR_                        388
```
The STA Reports can be viewed from the Reports folder.

The openSTA tool generated the timing reports. It can be seen from below that 

### Key concepts

#### Utilisation Factor 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1613/14876 = 0.1084** (i.e: 10.8%) [From the synthesis statistics]

#### Utilisation Factor

- The ratio of area occupied by the cells in the netlist to the total area of the core
- Best practice is to set the utilisation factor less than 50% so that there will be space for optimisations, routing, inserting buffers etc.,

### Aspect Ratio

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

 
### Cell Design Flow

 In a border view Cell Design flow is are the stages or steps involved in the entire design of a standard cell. The figure below shows the input, output and design steps involved in cell design
  
 
### Characterization Flow
  There are few problems of Standard Cells in polygon level format (GDSII). Some of them are:
  - Extraction of functionality is complicated and unnecessary as it is known
  - Functional/Delay simulation takes way too long
  - Power extraction for a whole chip takes too long
  - Automatic detection of timing constraints (e.g. Setup time) is difficult

  A solution to above problems is Cell Characterization. It is a simple model for delay, function, constraints and power on cell/gate level. The Characterization Flow consists of the following stages:
  1. Netlist Extraction - Transistors, resistances and capacitances are extracted with special tools and saved as SPICE netlist (or similar)
  2. Specification of parameters - Library-wide parameters have to be specified: e.g. max Transition time
  3. Model selection and specification - The used models determine the required data
  4. Measurement - The cells are simulated with a SPICE-like tool to obtain the required data
  5. Model Generation - The obtained data is fed into the models
  6. Verification - Different checks are performed to ensure the correctness of the characterization
  
  
- Inputs : PDK, DRC & LVS rules, SPICE models, library & User defined specs
    - The introduction of lambda based design rules allowed a design to be loosely tied with the fabrication process
    - The layout geometry (DRC) are expressed in terms of multiples of lambda which is half the feature size
    - Users define the cell height to be the separation between the power and the ground rail
    - Cell width is dependent on the timing information and required drive strength
    - Cell Width increases, Area Increases, Timing decreases, Drive Strength increases as the Resistance and Capacitance decreases(RC)
    - Supply voltage is also specified by the top level design
    - The designed cell must fit in the above specifications
- Output : CDL(Circuit Description Language), GDSII(Graphic Design Standard 2), LEF(Layout Exchange Format), .lib containing Timing, Noise and Power characteristics


The inverter design is done using Magic Layout Tool. It takes the technology file as an input (`sky130A.tech` in this case). Magic tool provide a very easy to use interface to design various layers of the layout. It also has an in-built DRC check fetaure.

The snippet below shows a layout for CMOS Inverter with and without design rule violations.
  
![Screenshot from 2022-08-21 14-38-06](https://user-images.githubusercontent.com/88897605/185787840-d52b173d-f8ec-47b4-8217-f630ada843d3.png)
  
* Post Layout Waveform

![inverter_ngspice](https://user-images.githubusercontent.com/88897605/185787832-ad07de39-619e-4d86-982a-d110ab17af8c.jpeg)







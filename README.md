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




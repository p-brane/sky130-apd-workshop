# Skywater 130n Advanced Process Design Workshop

![](assets/README-dff83cda.png)
Image credit: efabless post on LinkedIn

efabless sponsored a [workshop](https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/) on Advanced Physical Design using the openLANE open source toolchain, and the Skywater 130 nm Physical Design Kit (PDK). Google and Skywater have been working to make custom semiconductor design and processing open source. The course was produced and taught by VLSI System Design (VSD) -  Intelligent Assessment Technology (IAT).

The workshop was held August 2 to 6 and each day begins with an Q&A session at 10:30 AM ET over Zoom. The course lectures were recorded and made available on the VSD learning platform. The platform also provided a compute instance containing all the tools and PDK library all setup to use.

The platform was very responsive for both the lecture recordings and running the labs to learn the design toolchain. The workshop also provided a Slack workspace to interact with other students and get questions answered. The instructors were very good about responding to questions to help solve problems with simulations. The subjects covered each day are shown below. Each module also had exams.

* Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
* Day 2 - Good floorplan vs bad floorplan and introduction to library cells
* Day 3 - Design library cell using Magic Layout and ngspice characterization
* Day 4 - Pre-layout timing analysis and importance of good clock tree
* Day 5 - Final steps for RTL2GDS using tritonRoute and openSTA

The first three days introduce you to the openLANE toolchain and the PDK. On the last two days, we replaced a standard cell inverter with a custom cell in the picorv32a design to demonstrate how to add a custom cell to an existing design. The picorv32a is a [Size-Optimized RISC-V CPU](https://github.com/YosysHQ/picorv32) core that executes the RISC-V RV32IMC Instruction Set. The sections below describe show my results produced from the workshop labs. The workshop is intense and required around 10 to 12 hrs/day to work through the lectures, labs and exams as this was my first introduction to an ASIC design toolchain.

There wasn't enough time to finish the timing optimization. Fortunately, the tools can be downloaded and installed on a local computer and completed offline. Designs can be submitted to the Google/Skywater Multi-Project Wafer (MPW) shuttle. The submission deadline is Sept 12, 2022.

## Day 1

OpenLANE integrates a tool flow into a system that takes in a design and a PDK, and generated a GDSII output. The process can be automated or interactive. The figure below shows the tools and the tool flow for openLANE.

![](assets/README-7a918a51.png)
Image credit: [Openlane-docs: Openlane Acrheture](https://openlane-docs.readthedocs.io/en/rtd-develop/#openlane-architecture)

The design flow sequence used for the labs and openLANE is summarized below and was followed up trough magic layout.

* synthesis
* floorplan
* placement
* clock tree synthesis
* routing
* magic layout
* magic spice export
* magic DRC
* netgen DRC
* magic antenna check

### Start openLANE

The following sequence was used to run an interactive OpenLANE flow to synthesis the picorv32a design.

```
cd ~/Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
The figure below shows the output of the first step.

* ![](assets/sky130_pd_lab_01_noted-da8234e6.png)
* ![](assets/sky130_pd_lab_01_noted-09e8873b.png)

### Synthesis

The figure below show the end of the output from `run_synthesis`

* ![](assets/sky130_pd_lab_01_noted-5d72c0ab.png)

The Flop ratio from 1-yosys_4.stat.rpt is 1613/14876=0.1084

The buffer (buf1) ratio is 1656/14876=0.1113

## Day 2

### Floorplan Generation
The next step is to create a floorplan for the picorv32a design.

`run_floorplan`

The details of the configuration variable are located in `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/configuration/README.md??????` under the Floorplanning section. Each variable is shown, defined, and a default value is defined. Some of the floorplanning variable are shown in the figure below.

![](assets/sky130_apd_workshop_day2_lab1_results-0793bd3d.png)

The default values can be changed in the floorplan.tcl file. Some of the floorplan.tcl is shown in the figure below.

![](assets/sky130_apd_workshop_day2_lab1_results-f2256850.png)

The floorplan.tch file places the horizontal metal on layer 3 which is metal4 and the vertical metal on layer 4 which is metal5. The metal layer stack is shown in the figure below.

![](assets/sky130_apd_workshop_day2_lab1_results-6f24db54.png)
Image credit: [Skywater PDK Process Stack Diagram](https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html#process-stack-diagram)

The `run_floorplan` runs for a while and indicate a successful completion.

![](assets/sky130_apd_workshop_day2_lab1_results-20b5e7e9.png)

The log file `ioPlaer.log` from the floorplan are found at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-08_02-33/logs/floorplan/`

![](assets/sky130_apd_workshop_day2_lab1_results-c604f3c5.png)

The `ioPlacer.log` file contains the following info.

```
??????OpenROAD 0.9.0 1415572a73
This program is licensed under the BSD-3 license. See the LICENSE file for details.
Components of this program may be licensed under more restrictive licenses which must be honored.
Notice 0: Reading LEF file:  /openLANE_flow/designs/picorv32a/runs/03-08_02-33/tmp/merged.lef
Notice 0:     Created 13 technology layers
Notice 0:     Created 25 technology vias
Notice 0:     Created 440 library cells
Notice 0: Finished LEF file:  /openLANE_flow/designs/picorv32a/runs/03-08_02-33/tmp/merged.lef
Notice 0:
Reading DEF file: /openLANE_flow/designs/picorv32a/runs/03-08_02-33/tmp/floorplan/3-verilog2def_openroad.def
Notice 0: Design: picorv32a
Notice 0:     Created 409 pins.
Notice 0:     Created 14876 components and 115597 component-terminals.
Notice 0:     Created 14978 nets and 56051 connections.
Notice 0: Finished DEF file: /openLANE_flow/designs/picorv32a/runs/03-08_02-33/tmp/floorplan/3-verilog2def_openroad.def
#Macro blocks found: 0
Using 5u default boundaries offset
Random pin placement
RandomMode Even
```

The default `config.tcl` and default sky130A_sky130_fd_sc_hd_config.tcl file were used for this floorplan run. The configuration files are located at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/`

![](assets/sky130_apd_workshop_day2_lab1_results-70d94a3c.png)

The `picov32a.def` file contains the floorplan results and is located at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-08_06-01/results/floorplan/`. The total die area is 660685 by 671405 units and can be converter to microns by dividing 1000 (1 micron = 1000 units).

![](assets/sky130_apd_workshop_day2_lab1_results-9b1ceaef.png)

### Floorplan Layout with Magic

`Magic` will display the floorplan layout graphically. `Magic` is invoked as using the following command in a new terminal.

`magic -T /home/p-brane/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-08_06-01/tmp/merged.lef def read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-08_06-01/results/floorplan/picov32a.floorplan.def &`

The `sky130A.tech` file describes the Skywater 130 nm process and the `picov32a.floorplan.def` file contain the floorplan for the picorv32a design generated by the `run_floorplan` command.

![](assets/sky130_apd_workshop_day2_lab1_results-e2cbacff.png)
![](assets/sky130_apd_workshop_day2_lab1_results-8e63b358.png)

The Layout appears.
![](assets/sky130_apd_workshop_day2_lab1_results-d4123476.png)
![](assets/sky130_apd_workshop_day2_lab1_results-d33013ea.png)

To zoom, left mouse click and then right mouse click to form a rectangle and press `z` to zoom. Hover over a cell and press 's' to select and the `tkcon` window will show the status of the cell. The standard cells are located in the lower left corner.

![](assets/sky130_apd_workshop_day2_lab1_results-57dfeee4.png)

An error occurred so `run_synthesis` and `run_placement` error run again.

![](assets/sky130_apd_workshop_day2_lab1_results-9c08a1e3.png)

The `Magic` command becomes

`magic -T /home/p-brane/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/tmp/merged.lef def read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57j/results/floorplan/picorv32a.floorplan.def &`

`Magic` now shows the status of the standard cells, there may still be an error.

![](assets/sky130_apd_workshop_day2_lab1_results-5ed1fb95.png)

### Placement

To place the standard cells run 'run_placement'.

![](assets/sky130_apd_workshop_day2_lab1_results-83a009ec.png)
![](assets/sky130_apd_workshop_day2_lab1_results-f8b39466.png)

## Day 3

### CMOS Inverter Standard Cell

Download the `vsdstdcelldesign` from GitHub to the openlane directory. This can be accomplished using the following commands

`cd /home/Desktop/work/tools/openlane_working_dir/openlane`
`git clone https://github.com/nickson-jose/vsdstdcelldesign.git`

![](assets/sky130_apd_workshop_day3_lab1_results-34b60ffb.png)

Move into the `vsdstdcelldesign` directory and copy the sky130A.tech file to the `vsdstdcelldesign` directory

`cd vsdstdcelldesign`
`cp /home/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sly130A.tech  .`

![](assets/sky130_apd_workshop_day3_lab1_results-2000f042.png)

### Inverter Layout in Magic

Run `Magic` using `magic -d XR -T sky130A.tech sky130_inv.mag&` The -d XR setting use a different graphics system and produces a clearer image.

![](assets/sky130_apd_workshop_day3_lab1_results-b1507618.png)

### Extracted SPICE Netlist

To extract a spice netlist, use the `extract all` and `ext2spice' commands as shown below.

```
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

![](assets/sky130_apd_workshop_day3_lab1_results-810208a2.png)

This generates the `sky130_inv.ext` and `sky130_inv.spice` files.

![](assets/sky130_apd_workshop_day3_lab1_results-2431da87.png)

`sky130_inv.ext` file

```
timestamp 1600540370
version 8.3
tech sky130A
style ngspice()
scale 1000 1 1e+06
resistclasses 2200000 3050000 1700000 3050000 120000 197000 114000 191000 120000 197000 114000 191000 48200 319800 2000000 48200 48200 12200 125 125 47 47 29 5
parameters sky130_fd_pr__nfet_01v8 l=l w=w a1=as p1=ps a2=ad p2=pd
parameters sky130_fd_pr__pfet_01v8 l=l w=w a1=as p1=ps a2=ad p2=pd
port "Y" 2 96 121 131 164 li
port "A" 1 10 121 45 164 li
port "VPWR" 3 44 258 90 289 m1
port "VGND" 5 50 -3 105 26 m1
node "Y" 474 347.122 96 121 li 0 0 0 0 0 0 0 0 1435 152 1443 152 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 5951 544 0 0 0 0 0 0 0 0 0 0 0 0
node "A" 462 409.094 10 121 li 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 7322 578 0 0 2295 192 0 0 0 0 0 0 0 0 0 0 0 0
node "VPWR" 2717 828.259 44 258 m1 0 0 0 0 33630 734 0 0 2950 286 1517 156 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6865 546 8496 450 0 0 0 0 0 0 0 0 0 0
substrate "VGND" 0 0 50 -3 m1 0 0 0 0 0 0 0 0 1365 148 2574 278 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6941 498 8064 432 0 0 0 0 0 0 0 0 0 0
cap "VPWR" "A" 66.0605
cap "VPWR" "Y" 106.092
cap "A" "Y" 45.4865
device msubckt sky130_fd_pr__nfet_01v8 49 41 50 42 l=23 w=35 "VGND" "A" 46 0 "VGND" 35 0 "Y" 35 0
device msubckt sky130_fd_pr__pfet_01v8 49 196 50 197 l=23 w=37 "VPWR" "A" 46 0 "VPWR" 37 0 "Y" 37 0
subcap "Y" -107.681
subcap "A" -45.9469
subcap "VPWR" -243.219
subcap "VGND" -109.355
```

`sky130_inv.spice` file

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=10000u

.subckt sky130_inv A Y VPWR VGND
X0 Y A VGND VGND sky130_fd_pr__nfet_01v8 ad=0 pd=0 as=0 ps=0 w=35 l=23
X1 Y A VPWR VPWR sky130_fd_pr__pfet_01v8 ad=0 pd=0 as=0 ps=0 w=37 l=23
C0 Y A 0.05fF
C1 VPWR A 0.07fF
C2 Y VPWR 0.11fF
C3 Y VGND 0.24fF
C4 VPWR VGND 0.59fF
.ends
```
### Spice Deck

Update the `sky130_inv.spice` file to use the correct scaling factor and models. From the `tkcon` the root cell box is 0.01 x 0.01 microns. The scale is change om the `.option` command as follows: `.option scale =0.01u'

![](assets/sky130_apd_workshop_day3_lab1_results-c74458c3.png)

Updated SPICE Netlist

```
??????* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/nshort.lib
.include ./libs/pshort.lib

//.subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=0 pd=0 as=0 ps=0 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=0 pd=0 as=0 ps=0 w=37 l=23
VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1n 0.1n 2ms 4m)
C0 Y A 0.05fF
C1 VPWR A 0.07fF
C2 Y VPWR 0.11fF
C3 Y VGND 0.24fF
C4 VPWR VGND 0.59fF
//.ends
.tran 1n 20m
.control
run
.endc
.end
```
### SPICE Simulation

Simulate the SPICE deck using 'ngspice sky130_inv.spice'

![](assets/sky130_apd_workshop_day3_lab1_results-099cefbf.png)

Then plot the results `plot y vs time a`

![](assets/sky130_apd_workshop_day3_lab1_results-d200084f.png)

The rise and fall times, and propagation delay can be then found from the plot. This measurement wasn't performed due to time limitations.

## Day 4

### Inverter Cell

Go to the `vsdstdcelldesign` directory at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign` and run `Magic` on the for the inverter.

???`magic -d XR -T sky130A.tech sky130_inv.mag&`

![](assets/sky130_apd_workshop_day4_lab1_results-0b8b6d56.png)

![](assets/sky130_apd_workshop_day4_lab1_results-e46a7ca3.png)

### Check Grid Alignment
The inverter cell is laid out on a grid that must match the pitch and offsets defined in the tracks.info file located at `/home/p-brane/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.inf`
The file contains the layer name, X or Y dimension, offset, and pitch as shown below.
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
The default grid is 0.01 um x 0.01 um as shown in the `tkcon` above. It can be changed to match the `tracks.info` grid for the li (locali) layer. Using `grid 0.46um 0.34ym 0.23um 0.17um`

![](assets/sky130_apd_workshop_day4_lab1_results-1d1145c2.png)

The alignment of the cell elements with the grid can be verified.

![](assets/sky130_apd_workshop_day4_lab1_results-d5201aed.png)

### Assign Ports for LEF File Generation

To create a `lef` file, the ports of the inverter need to be defined as pins in the `lef` macro.

![](assets/sky130_apd_workshop_day4_lab1_results-b04a1f5d.png)

Define the input port A and set the `class` to input and `use` to signal.

![](assets/sky130_apd_workshop_day4_lab1_results-549c9921.png)
![](assets/sky130_apd_workshop_day4_lab1_results-a915e874.png)

Define output port Y and set the `class` to output and `use` to signal.

![](assets/sky130_apd_workshop_day4_lab1_results-7cdebd23.png)
![](assets/sky130_apd_workshop_day4_lab1_results-01e6c032.png)

Define VPWR port and set the `class` to input and `use` to power.

![](assets/sky130_apd_workshop_day4_lab1_results-295a3862.png)
![](assets/sky130_apd_workshop_day4_lab1_results-840c3dca.png)

Define VGND port and set the `class` to input and `use` to ground.

![](assets/sky130_apd_workshop_day4_lab1_results-42734cd2.png)
![](assets/sky130_apd_workshop_day4_lab1_results-19bf55d3.png)

Save the file: `save sky130_jayinv.mag`
![](assets/sky130_apd_workshop_day4_lab1_results-aa747cf7.png)

Open the new file using `magic -d XR -T sky130A.tech sky130_jayinv.mag&`

### LEF File

Use `lef write` to generate a `lef` file

![](assets/sky130_apd_workshop_day4_lab1_results-3b3b5d51.png)

The generated `sky130_jayinv.lef` file is shown below
```
VERSION 5.7 ;
  NOWIREEXTENSIONATPIN ON ;
  DIVIDERCHAR "/" ;
  BUSBITCHARS "[]" ;
MACRO sky130_jayinv
  CLASS CORE ;
  FOREIGN sky130_jayinv ;
  ORIGIN 0.000 0.000 ;
  SIZE 1.380 BY 2.720 ;
  SITE unithd ;
  PIN A
    DIRECTION INPUT ;
    USE SIGNAL ;
    ANTENNAGATEAREA 0.165600 ;
    PORT
      LAYER li1 ;
        RECT 0.060 1.180 0.510 1.690 ;
    END
  END A
  PIN Y
    DIRECTION OUTPUT ;
    USE SIGNAL ;
    ANTENNADIFFAREA 0.287800 ;
    PORT
      LAYER li1 ;
        RECT 0.760 1.960 1.100 2.330 ;
        RECT 0.880 1.690 1.050 1.960 ;
        RECT 0.880 1.180 1.330 1.690 ;
        RECT 0.880 0.760 1.050 1.180 ;
        RECT 0.780 0.410 1.130 0.760 ;
    END
  END Y
  PIN VPWR
    DIRECTION INPUT ;
    USE POWER ;
    PORT
      LAYER nwell ;
        RECT -0.200 1.140 1.570 3.040 ;
      LAYER li1 ;
        RECT -0.200 2.580 1.430 2.900 ;
        RECT 0.180 2.330 0.350 2.580 ;
        RECT 0.100 1.970 0.440 2.330 ;
      LAYER mcon ;
        RECT 0.230 2.640 0.400 2.810 ;
        RECT 1.000 2.650 1.170 2.820 ;
      LAYER met1 ;
        RECT -0.200 2.480 1.570 2.960 ;
    END
  END VPWR
  PIN VGND
    DIRECTION INPUT ;
    USE GROUND ;
    PORT
      LAYER li1 ;
        RECT 0.100 0.410 0.450 0.760 ;
        RECT 0.150 0.210 0.380 0.410 ;
        RECT 0.000 -0.150 1.460 0.210 ;
      LAYER mcon ;
        RECT 0.210 -0.090 0.380 0.080 ;
        RECT 1.050 -0.090 1.220 0.080 ;
      LAYER met1 ;
        RECT -0.110 -0.240 1.570 0.240 ;
    END
  END VGND
END sky130_jayinv
END LIBRARY
```

### Static Timing Analysis File Setup

In this lab, the `sky130_jayinv` will replace the standard cell inverter used in the picorv32a design to demonstrate how to add a custom cell to and existing design. The picorv32a is a [Size-Optimized RISC-V CPU](https://github.com/YosysHQ/picorv32) core that executes the RISC-V RV32IMC Instruction Set.

Copy the provided `my_base.sdc` file from the `vsdstdcelldesign/extras` directory and the `sky130_jayinv.lef` to the `picorv32a/src` directory. Copy the `vsdstdcelldesign/extras/sta.conf` file to the openlane directory.

```
cd /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/extras/
cp my_base.sdc /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
cp sta.conf /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/
cd /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/
cp sky130_jayinv.lef /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```
![](assets/sky130_apd_workshop_day4_lab1_results-0e84de67.png)

Copy the sky130 libraries to the `picrv32a/src` directory

```
cd /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/libs/
cp sky130_fd_sc_hd__* /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```
![](assets/sky130_apd_workshop_day4_lab1_results-8534a189.png)

### Config.tcl

`config.tcl` is used to configure openlane to use the `picorv32a` design, the `sky130_fd_sc_hd` libraries, define the clock and clock period, and use custom cells. Modify the `config.tcl` to include the copied libraries at  `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/config.tcl`
The modified file is shown below. `LIB_MIN` was replaced with `LIB_FASTEST`, and `LIB_MAX` was replace with `LIB_SLOWEST`.

```
# Design
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) "./designs/picorv32a/src/picorv32a.v"
set ::env(SDC_FILE) "./designs/picorv32a/src/picorv32a.sdc"

set ::env(CLOCK_PERIOD) "12.000"
set ::env(CLOCK_PORT) "clk"


set ::env(CLOCK_NET) $::env(CLOCK_PORT)

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]


set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
	source $filename
}
```

### pre_sta.conf

The `pre_sta.conf` static timing configuration file is used to configure the Static Timing Analysis tool STA. `sta pre_sta.conf` are used in another terminal. `sta pre_sta.conf` must re-run every time a new `run_synthesis` is performed so that `sta` analyses the changes to the synthesis file `picorv32.synthesis.v`.

`cd /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/` and rename the `sta.conf` file to `pre-sta.conf` using `mv sta.conf pre_sta.conf`. Edit the pre_sta.conf file as shown below to add the links to the libraries and the verilog file.
```
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
read_liberty -min /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_liberty -max /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_verilog /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/results/synthesis/picorv32a.synthesis.v
link_design picorv32a
read_sdc /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
report_checks -path_delay min_max -fields {slew trans net cap input_pin} -digits 4
report_tns
report_wns
```
### my_base.sdc

The `my-base.sdc` file is used to configure variables in the design, and is referenced by the `pre_sta.conf` file. The definition and default values are described in the `~/openlane/configuration/README.md` file.

```
set ::env(CLOCK_PORT) clk
set ::env(CLOCK_PERIOD) 12.000
set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv_8
set ::env(SYNTH_DRIVING_CELL_PIN) Y
set ::env(SYNTH_CAP_LOAD) 17.65
create_clock [get_ports $::env(CLOCK_PORT)]  -name $::env(CLOCK_PORT)  -period $::env(CLOCK_PERIOD)
set IO_PCT  0.2
set input_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
set output_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
puts "\[INFO\]: Setting output delay to: $output_delay_value"
puts "\[INFO\]: Setting input delay to: $input_delay_value"


set clk_indx [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
#set rst_indx [lsearch [all_inputs] [get_port resetn]]
set all_inputs_wo_clk [lreplace [all_inputs] $clk_indx $clk_indx]
#set all_inputs_wo_clk_rst [lreplace $all_inputs_wo_clk $rst_indx $rst_indx]
set all_inputs_wo_clk_rst $all_inputs_wo_clk


# correct resetn
set_input_delay $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
#set_input_delay 0.0 -clock [get_clocks $::env(CLOCK_PORT)] {resetn}
set_output_delay $output_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

# TODO set this as parameter
set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) -pin $::env(SYNTH_DRIVING_CELL_PIN) [all_inputs]
set cap_load [expr $::env(SYNTH_CAP_LOAD) / 1000.0]
puts "\[INFO\]: Setting load to: $cap_load"
set_load  $cap_load [all_outputs]
```

### OpenLANE

The files supplied from the [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign) repository contain a custom inverter that will be used in the `picorv32a` design. The inverter is called `sky130_vsdinv` in the libraries. If a different name is used then the `sky130_vsdinv` macro must be renamed in the libraries so that the `run_synthesis` tools can find the `sky130_jayinv` as I have named it.

To use the sky130_jayinv inverter, The libraries `sky130_fd_sc_hd__typical.lib`, `sky130_fd_sc_hd__slow.lib`, and `sky130_fd_sc_hd__fast.lib` in the src folder were modified with the new name `sky130_vsdinv` -> `sky130_jayinv`.

The lectures use `docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) openlane:rc2` to start docker. I was not able to get this to work properly and find that `docker` seems to work fine. Once `docker` is running then the follow commands can be issued to setup openlane for synthesis. The last two commands in the sequence enable openlane to use the custom inverter.

```
docker
./flow.tcl interactive
package require openlane 0.9
prep -design picorv32a -tag 04-08_06-57 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

![](assets/sky130_apd_workshop_day4_lab1_results-3cc07023.png)

### Synthesis

`run_synthesis` produces the following output.

![](assets/sky130_apd_workshop_day4_lab1_results-b66c6a0b.png)

### Static Timing Analysis

Static timing analysis can be performed using `sta pre-sta.conf` from a terminal in the `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/` directory. The setup slack violation at this point is -36.62.

![](assets/sky130_apd_workshop_day4_lab1_results-cbd1d8d2.png)

The hold slack is 0.24 and is not in violation.

![](assets/sky130_apd_workshop_day4_lab1_results-e09e85b8.png)

### Timing Violation Reduction

`my_base.sdc` can be updated to change default settings on variables used for the synthesis. The input capacitance of the `sky130_fd_sc_hd__inv_8` is 17.65 pF as defined in the `sky130_fd_sc_hd__typical.lib` library. This value was used to seet the `SYNTH_CAP_LOAD` value is the `my_base.sdc`. These value should be verified and updated when a different library is used and when trying to reduce timing violations.

![](assets/sky130_apd_workshop_day4_lab1_results-cb2a6a27.png)

Synthesis variable values can be read and can be changed interactively using the set command instead of modifying the `my_base.sdc` file. Some examples are shown below.

```
set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv.8
set ::env(SYNTH_DRIVING_CELL_PIN) Y
set ::env(SYNTH_CAP_LOAD) 17.6
set ::env(SYNTH_MAX_FANOUT) 4
```

The `SYNTH_MAX_FANOUT` defines the gate fanout. Reduing the fanout should reduce the slack. `echo $::env(SYNTH_MAX_FANOUT)` shows that the default fanout is 6. The fanout `set ::env(SYNTH_MAX_FANOUT) 4` was used to set the fanout to 4.

![](assets/sky130_apd_workshop_day4_lab1_results-42650f3d.png)

Reducing the fanout did not reduce the slack. Further, work will be needed to reduce the slack so that it is zero or positive. The process involves changing gates on slow nets by replacing cells using the `replace_cell instance lib_cell` command in STA, rerunning synthesis (`run_synthesis`), and re-running `sta pre_sta.conf` to see the changes in slack timing violations (if any).

In STA, net `_11643_` has a high delay. It is driven with a `sky130_fd_sc_hd__or4_2` gate. To reduce the delay it will be replaced with the `sky130_fd_sc_hd__or4_4`which is larger and should be faster. The cell is replaced using `replace_cell _14481_ sky130_fd_sc_hd__or4_4`.

![](assets/sky130_apd_workshop_day4_lab1_results-6f6df258.png)

![](assets/sky130_apd_workshop_day4_lab1_results-549c9486.png)

This reduced the slack slightly.

![](assets/sky130_apd_workshop_day4_lab1_results-5c6a6b70.png)

When the timing slack has been reduced to zero the interactively modified Verilog file can be written to a file and used in following steps. In STA, use `write verilog /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/results/synthesis/picorv32a.synthesis.v` to save the file. It would be wise to backup the file before writing over it.

### Floorplan

`run_floorplan` failed due not finding a source file.

![](assets/sky130_apd_workshop_day4_lab1_results-24ecd312.png)

The following procedure was used to manually create a floorplan.

```
init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement
gen_pdn
run_routing
```
`init_floorplan`
![](assets/sky130_apd_workshop_day4_lab1_results-446a4372.png)
`place_io`
![](assets/sky130_apd_workshop_day4_lab1_results-93214a69.png)

`global_placement_or`
![](assets/sky130_apd_workshop_day4_lab1_results-10021b96.png)

`detailed_placement`
![](assets/sky130_apd_workshop_day4_lab1_results-fe36af1e.png)

`tap_decap_or`
![](assets/sky130_apd_workshop_day4_lab1_results-60c36546.png)

`detailed_placement`
![](assets/sky130_apd_workshop_day4_lab1_results-b354f240.png)

`gen_pdn`
![](assets/sky130_apd_workshop_day4_lab1_results-7559b361.png)

### Magic

`Magic` was used to view the placement of the design using the command below.
```
magic -d XR -T /home/p-brane/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/tmp/merged.lef def read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/results/placement/picorv32a.placement.def&
```
![](assets/sky130_apd_workshop_day4_lab1_results-96f2ad87.png)

Finding the `skay130_jayinv ` cell turn out to be more difficult that I expected. Using the `s` I selected it and used `what` to lean more about it. `expand` was used to show the layout of the cell!

![](assets/sky130_apd_workshop_day4_lab1_results-cb0405f8.png)

### Clock Tree Synthesis (CTS)

`run_cts` starts the clock tree synthesis and it completed successfully, and has a slack violation. The slack violation are now much lower at -0.2 ns compared with the slack violation of -32.62 ns reported from the initial static timing analysis. `echo $::env(CTS_TOLERANCE)` can be used to display the Quality of Results (QoR) factor used for the CTS run. The default value is 100. Higher values will produce lower slack values and will require more time to process. OpenROAD and openSTA are used to analyze the timing.

![](assets/sky130_apd_workshop_day4_lab1_results-22c2822e.png)
![](assets/sky130_apd_workshop_day4_lab1_results-3cabfcf3.png)

The `run_cts` configuration file `cts.tcl` is located at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/scripts/tcl_commands/` and defines the clock nets and clock timing related variables.

# OpenRoad and OpenSTA

The OpenROAD and OpenSTA timing analysis labs weren't completed due to time limitations.

## Day 5

### Power Distribution Network (PDN)

The placement passed all legality checks. Then the `gen_pdn` command was used to generate the PDN.

![](assets/sky130_apd_workshop_day5_lab1_results-81701dc6.png)

![](assets/sky130_apd_workshop_day5_lab1_results-fcb41129.png)

### Routing

The `run_routing` command was used to generate the routing using the default settings. The routing completed with thee violations after running for 17.5 minutes.

![](assets/sky130_apd_workshop_day5_lab1_results-4cb468ef.png)

The `20-tritonRouting.drc` log file can be found at `/home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/reports/routing/` and shows the DRC violations. These are shown below.

```
violation type: Short
  srcs: frFakeVSS _05946_
  bbox = ( 559.19, 470.32 ) - ( 559.33, 470.8 ) on Layer met1
violation type: Short
  srcs: VGND _05946_
  bbox = ( 559.19, 470.32 ) - ( 559.33, 470.8 ) on Layer met1
violation type: Short
  srcs: VGND _05657_
  bbox = ( 559.98, 470.32 ) - ( 560.12, 470.8 ) on Layer met2
```

### Magic

The following command was used to display the design with routing but without the CTS in Magic.

```
magic -d XR -T /home/p-brane/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/tmp/merged.lef def read /home/p-brane/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/04-08_06-57/results/routing/picorv32a.def&
```

![](assets/sky130_apd_workshop_day5_lab1_results-934102a2.png)

`sky130_fd_sc_hd__buf_1` was selected and expanded to better show the routing.

![](assets/sky130_apd_workshop_day5_lab1_results-4ab33ad8.png)

Magic was used again to display the design for the design with CTS followed by routing. The routing here has no DRC violations and a slack violation of -0.2 ns. This can be further improved iteratively with gate replacements and STA analysis.

![](assets/sky130_apd_workshop_day5_lab1_results-c847cb2e.png)

'sky130_jayinv' was expanded to show the cell layout with the routing.
![](assets/sky130_apd_workshop_day5_lab1_results-042ac58b.png)

### Parasitic Extraction

Parasitic RC extraction can be performed on the routed design using the python3 program `SPEF_EXTRACTOR`. The extraction is combined with the routed `picorv32a.def` file to check for timing errors due to parasitics. The parasitic extraction flow wasn't performed due to time limitations.

### References

* [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)
* [OpenLANE Workshop](https://gitlab.com/gab13c/openlane-workshop)
* [OpenLANE Docs](https://openlane-docs.readthedocs.io/en/rtd-develop/#openlane-architecture)
* [Skywater SKY130 PDK](https://antmicro-skywater-pdk-docs.readthedocs.io/en/latest/index.html)
* [SkyWater / Google / Efabless SKY130 OpenMPW Resources](https://github.com/efabless/skywater-pdk-central)
* [PicoRV32 - A Size-Optimized RISC-V CPU](https://github.com/YosysHQ/picorv32)
* [Standard cell design and characterization using openlane flow](https://github.com/nickson-jose/vsdstdcelldesign)
* [Magic VLSI Layout Tool](http://opencircuitdesign.com/magic/)
[Magic User's Guide](http://opencircuitdesign.com/magic/commandref/commands.html)
[Magic GitHub](https://github.com/RTimothyEdwards/magic)
* [ngspice](https://ngspice.sourceforge.io/spdevs.html)

## Certificate of Completion

![](assets/2022_VSD-IAT_Workshop_Certificate_jay_morreale2.jpg)

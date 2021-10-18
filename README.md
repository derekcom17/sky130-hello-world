# sky130-hello-world

The example is based around the minimum-size inverter from the SKY130 PDK.  DRC is run on the GDS for that inverter, and then it is extracted for LVS and PEX.  On the LVS front, the extracted SPICE netlist is compared to the netlist given in the PDK using ``netgen``.  On the PEX front, ``ngspice`` is used to run a transient simulation of the inverter with parasitic capacitances included.  The regression tests also include some tests to make sure that DRC and LVS errors can in fact be detected, using the intentionally broken file ``inv1_bad.gds``.

The full details for this example can always be found by examining the source code here, but the main functions are also summarized below.

## Installing dependencies

Certain dependencies need to be installed before moving on to the SKY130-specific tools.

```shell
sudo apt-get install m4 tcsh csh libx11-dev tcl-dev tk-dev libcairo2-dev libncurses-dev libglu1-mesa-dev freeglut3-dev mesa-common-dev ngspice
```

### MAGIC VLSI Layout
Magic is a dependency to install `open_pdks`. 

Installation instructions: http://opencircuitdesign.com/magic/

Source found here: https://github.com/RTimothyEdwards/magic

### PDK
Next, The sky130 PDK must be installed.

This PDK is installed via `open_pdks`.

Installation instructions: http://opencircuitdesign.com/open_pdks/

Source found here: https://github.com/RTimothyEdwards/open_pdks

## Installing tools

Running `make install-tools` installs the following tools locally: ``magic``, ``netgen``, ``ngspice``, ``xschem``.

If the PDK install location was manually set, update the `PDKSPATH` and/or `PDKNAME` at the top of the Makefile before installing.

## Launching tools
Because the tools are not added to the shell path, Make targets were added as shourtcuts to launch them:

`make launch-xschem`

`make launch-ngspice`

`make launch-magic`

`make launch-netgen`


The following syntax is used to pass additional arguments into the programs:

`make launch-<TOOL> args=<ADDITIONAL_ARGS>`

For example: `make launch-ngspice args=--version` will launch NGSPICE, print the version, and exit.

## Running DRC

Launch Magic, and enter the following commands into the Magic console to count up the number of DRC violations.  For some reason, the final line says that there are zero DRC errors, even if previous lines show errors.

```
% gds read inv1.gds
% load sky130_fd_sc_hd__inv_1
% drc catchup
% drc count
```

To confirm the DRC check, you may replace the gds file in the first line with `inv1_bad.gds`. This file is expected to have DRC errors.

## Running PEX

Launch Magic, and enter the following commands into the Magic console to generate a netlist of the example cell with Parasitic Extraction. 

```
% gds read inv1.gds
% load sky130_fd_sc_hd__inv_1
% extract all
% select top cell
% port makeall
% ext2spice lvs
% ext2spice cthresh 0.01
% ext2spice rthresh 1
% ext2spice subcircuit on
% ext2spice -o pex_inv1.spice
```

The output of these commands will be `pex_inv1.spice`

## Running LVS

The following ``magic`` TCL script reads in a GDS and extracts it to an LVS-ready netlist.

```tcl
gds read path_to_gds_file
load name_of_top_cell
extract all
ext2spice lvs
ext2spice subcircuits off
ext2spice -o lvs_output_file
quit
```

LVS can be run to compare the design netlist with the extracted netlist using ``netgen``:

```shell
netgen -batch lvs "design_netlist design_top_cell" "ext_netlist ext_top_cell" $PDKPATH/libs.tech/netgen/sky130A_setup.tcl
```

## Running SPICE simulations

SPICE simulations can be run without special options (e.g., ``ngspice myfile.spice``), but the following line should be placed near the top of the input file.

```spice
.lib "skywater-pdk/libraries/sky130_fd_pr/latest/models/sky130.lib.spice" tt
```

You'll probably need to make that an absolute path unless ``skywater-pdk`` happens to be in your current direction.  Also, remember that for ngspice, the very first line of the input file must be a comment!

## Acknowledgements
* SKY130 inverter example with a video
  * https://github.com/mattvenn/magic-inverter
* SKY130 chip example with DRC/LVS/PEX scripts
  * https://github.com/yrrapt/amsat_txrx_ic
* SkyWater Open Source PDK
  * https://github.com/google/skywater-pdk
  * https://github.com/RTimothyEdwards/open_pdks
* Open Circuit Design Tools
  * https://github.com/RTimothyEdwards/magic
  * https://github.com/RTimothyEdwards/netgen


# Generating our own EIC events via Pythia8
As is specified in the [ddsim documentation](TODO), we can use centrally generated Pythia events instead of generating them ourselves.
If you'd rather generate them yourself, this is the tutorial for you.

To generate events with Pythia8, we have to pay attention to the reference frame.
EIC doesn't use events generated in the head-on collision frame of reference:
* EIC beams have a crossing angle of -0.025 mrad, and
* there is a beam energy smearing effect.

There are two options to work with these differences.
First, there is an afterburner that can correct generated events, which can be seen [here](https://github.com/eic/afterburner).
There's also steering code for Pythia8 that forces it to consider these two effects, which is the approach we'll use here.
After cloning the `eicSimuBeamEffects` from [here](https://github.com/eic/eicSimuBeamEffects) and running `make` in the `Pythia8` directory, wen can quickly generate EIC events by running
```bash
./runBeamShapeHepMC.exe steerFiles/dis_eicBeam_hiDiv_10x100 1 100 10 -0.025 \
<OUTPUTFILE>.hist.root <OUTPUTFILE>.hepmc
```
where the parameters passed to the function (in order) are:
* `1`: Configuration flag.
Set to 1 for hiDiv, 2 for hiAcc, and 3 for when running eA energies (18x110, 10x110, 5x110, and 5x41).
* `100`: Energy of the proton beam in GeV.
* `10`: Energy of the electron beam in GeV.
* `-0.025`: Crossing angle (x-angle) between the beams.
Note that this is in radians.
The coordinate system is such that z points in the direction of the hadron beam, y points to the sky, and x points to the ring center.
* `<OUTPUTFILE>.hist.root`: Output ROOT file with validation plots.
* `<OUTPUTFILE>.hepmc`: Output hepmc file to be used as input for `ddsim`.

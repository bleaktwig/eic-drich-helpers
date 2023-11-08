# Simulation with DDSim
## Single particle simulations
The program `ddsim` is used for simulations.
It has a miriad of options available to tune the simulation via command line, but it's recommended to use a python steering file for control.
To get a sample steering file, we can simply run
```bash
ddsim --dumpSteeringFile > sample_steering.py
```
The produced file even has a bunch of documentation on each of the options used.

The most commonly configured parameters for the simulation are
* `SIM.action.*` sets the actions of the different sensitive detectors,
* `SIM.field.*` configures the magnetic field,
* `SIM.filter.*` adds filters to the detectors,
* `SIM.gun.*` sets the single particle gun,
* `SIM.physics.*` allows setting the physics list, and
* `SIM.random.*` fixes the random seed.

To avoid confusion, it is useful to remove everything that we won't use from the steering file.
The most minimal steering file we can produce is
```python3
from DDSim.DD4hepSimulation import DD4hepSimulation
from g4units import mm, GeV, MeV
SIM = DD4hepSimulation()
```
which can be passed to `ddsim` using ``--steeringFile``.

If we actually run this file, we'll encounter some errors.
We need to setup a compact file, number of events, input file, particle gun, and userInputPlugin.
Via command line, we can set these properties via
```bash
ddsim --steeringFile <STEERING>.py --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml -G -N 10
#      \-> "empty" steering file.   \-> detector geometry.       use default gun. <-/    \-> set nevents.
```

For example, we can configure a 1 - 10 GeV pion+ gun with the following configuration in our steering file:
```python
# Throw uniformly on the unit sphere in the forward direction.
SIM.gun.distribution = "cos(theta)"

## The kinetic energy for the particle gun.
## NOTE. In the context of EIC, it's not recommended to use this option.
SIM.gun.energy = None

## Minimal and maximal momentum when using distribution (default = 0.0).
SIM.gun.momentumMin =  1*GeV
SIM.gun.momentumMax = 10*GeV

## Gun particle.
SIM.gun.particle = "pi+"

## Minimal and maximal polar angle for random distribution
SIM.gun.thetaMin =  3*deg
SIM.gun.thetaMax = 45*deg
```

## Output file handling
The output file name can be specified with the `--outputFile <FILENAME>` flag or the `SIM.outputFile   = <FILENAME>` line in the steering file.
`ddsim` will infer the output format based on the output file extension, and the current standard for the EIC collaboration is `.edm4hep.root`

After running the simulation, we can read the output file using ROOT
```bash
root -l <FILENAME>.root
```
Here, we can run `.ls` to see the trees in the file.
We'll focus on the `events` tree, which naturally contain the events data.

For the truth PDG of the generated particles, we can run
```bash
events->Draw("MCParticles.PDG", "MCParticles.generatorStatus == 1")
#             \-> Truth particles.  \-> Only accept particles that were thrown into the detector.
```
Or, for example, to see the deposited energy of the vertex barrel hits, we do
```bash
events->Draw("VertexBarrelHits.EDep")
```

## Running physics simulations
### Centrally produced input files
The large input files for simulation campaigns are stored on S3, which the `eic-shell` environment has access to.
We need to setup our access credentials to add the host, as in
```bash
mc config host add S3 https://eics3.sdcc.bnl.gov:9000 $S3_ACCESS_KEY $S3_SECRET_KEY
```
If you don't know the `$S3_ACCESS_KEY` and the `$S3_SECRET_KEY`, see the software tutorials or ask someone from the collaboration.

From here, we can download some 20.000 Pythia events with
```bash
mc head -n 20000 S3/eictest/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc > pythia8NCDIS_10x100.hepmc
```
and specify this HepMC3 input file as input to `ddsim`
```bash
ddsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles pythia8NCDIS_10x100.hepmc --outputFile pythia8NCDIS_10x100.edm4hep.root
```

These input files may also be requested on-demand from the EIC XRootD server, using
```bash
ddsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles root://dtn-eic.jlab.org//work/eic2/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc3.tree.root --outputFile pythia8NCDIS_10x100.edm4hep.root
```

### Generating our own input files
See the [hepmc3 documentation](hepmc3.md).

## NPSim vs. DDSim
Since dealing with the python steering file can be cumbersome, the `npsim` tool is provided as a layer on top of `ddsim`.
`npsim` has a bunch of options already specified, and has the following additional options:
* Cherenkov and optical photon physics are added through a python setup function,
* an optical photon filter is created and added to the dRICH,
* a modified tracking detector action which absorbs optical photon,
* the minimal energy deposition in the tracker is set to zero.

As should be obvious, `npsim` is the preferred option for dRICH event generation since optical photons are already included.
We can run it just as we would run `ddsim`.
E.g. to get 100 pythia events from the EIC Jlab server and simulate them on `ddsim` through `npsim`, we run
```bash
npsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles root://dtn-eic.jlab.org//work/eic2/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc3.tree.root --outputFile pythia8NCDIS_10x100.edm4hep.root
```
Naturally, it's much slower than `ddsim`, but we can see now that dRICH has hits.

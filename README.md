# EIC dRICH Helpers
This repository is just a small collection of markdown files that attempt to summarize the provided EIC tutorials with a focus on dRICH software development.
The current workflow in EIC software is shown in the following figure.
```
+-------------------------------------------------------------------------------------------------+
|                                                                                                 |
|  .-------------.                                                            .-----------------. |
|  |   hepmc3    |                                                        +-> |   podio file    | |
|  | (gen. data) |  __________                           ____________     |   | (reconstructed) | |
|  '-------+-----'  \         \      .--------------.    \           \    |   '-----------------' |
|          \-------> \  ddsim  \ --> |  podio file  | --> \ eicrecon  \ --/                       |
|          /-------> / (simul) /     | (simul data) |     /  (recon)  / --\                       |
|   .-----+------.  /_________/      '--------------'    /___________/    |   .---------------.   |
|   |   dd4hep   |                                                        +-> | eicrecon.root |   |
|   | (geometry) |                                                            | (user trees)  |   |
|   '------------'                                                            '---------------'   |
|                                                                                                 |
+-------------------------------------------------------------------+-----------------------------+
                                                                     \  EIC Singularity container |
                                                                      \___________________________|
```

## Relevant links
* [singularity](docs/singularity.md): Container program designed for HEP analysis, similar to docker.
* [hepmc3](docs/hepmc3.md):           Physics event generation.
* [dd4hep](docs/dd4hep.md):           EIC geometry definition.
* [ddsim & npsim](TODO):              Geant4 event simulation.
* [eicrecon](TODO):                   EIC event reconstruction.
* podio file format:                  TODO.

## Important notes
* The agreed-on EIC coordinate system is such that z points in the direction of the hadron beam, y points to the sky, and x points to the ring center.

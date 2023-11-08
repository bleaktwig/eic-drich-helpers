# EIC dRICH Helpers
This repository is just a small collection of markdown files that attempt to summarize the provided EIC tutorials with a focus on dRICH software development.
The current workflow in EIC software is shown in the following figure.
```
.-------------.                                                            .-----------------.
|   hepmc3    |                                                        +-> |   podio file    |
| (gen. data) |  __________                           ____________     |   | (reconstructed) |
'-------+-----'  \         \      .--------------.    \           \    |   '-----------------'
        \-------> \  ddsim  \ --> |  podio file  | --> \ eicrecon  \ --/
        /-------> / (simul) /     | (simul data) |     /  (recon)  / --\
 .-----+------.  /_________/      '--------------'    /___________/    |   .---------------.
 |   dd4hep   |                                                        +-> | eicrecon.root |
 | (geometry) |                                                            | (user trees)  |
 '------------'                                                            '---------------'
```

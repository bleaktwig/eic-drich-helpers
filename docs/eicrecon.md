# EICrecon
EICrecon is, as its name suggests, the program used for EIC reconstruction.
The package is already included in `eic-shell`, but if we want to make changes we need to clone the [git repository](https://github.com/eic/EICrecon), then make with
```bash
cmake -S . -B build
cmake --build build --target install -- -j4
```
and source
```bash
source bin/eicrecon-this.sh
```
to setup the path (and not take the recon installation from the shell).

There are two options to set parameters (which can be seen in the usage).
* `-Pkey=value` can be used to set configuration parameters in the job via the command line.
This option should only be used for a few parameters, since it can quickly become cumbersome.
* `-l <FILE>` is used to load configuration parameters from a file.
For an example file, you can add the flag `-d <FILE>` to dump the default configuration parameters into <FILE>.

***NOTE.*** *At the time of writing, there seems to be a bug with the `-d` option where it dumps an absurd amount of spaces on each line.*
*This can crash some text editors (like nano).*

To run reconstruction using the default parameters, we simply run
```bash
eicrecon <INPUTFILE>.edm4hep.root
```

This, however, won't write any output.
To get an output file, we need to tell `eicrecon` what to write
```bash
eicrecon -Ppodio:output_include_collections=ReconstructedParticles <INPUTFILE>.edm4hep.root
```
To see the list of possible collections to write, run `eicrecon -L`.

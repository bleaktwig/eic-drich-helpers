# Singularity
## Loading Singularity
```bash
## Load the singularity module from an HPC farm:
module load singularity

## Check that singularity works fine:
singularity --version
```

---
## Running images
### Images and containers
* **Images** are bundles of files including an OS, software, data, and other application-related files.
They can be stores either as a file or as a collection of files and have the `.sif` extension.
* **Containers** are virtual environments based on an image.
Whatever is available within a running container is determined by the image that the container is started from.

```bash
## Pull an image:
singularity pull hello_world.sif shub://vsoch/hello-world
#                 |               \-> where we're pulling the image from.
#                 \-> local file where we'll store the image.
```

### Running an image
Images come bundled with a run script.
* With `run`, we can run this script.
* With `exec`, we can ignore the run script and run our on stuff over the image.
* With `shell`, we get an interactive shell inside the image.

```bash
## Print an image's `run` script:
singularity inspect -r <IMAGE_NAME>.sif
#                       \-> image we're checking.

## Run an image's `run` script:
singularity run <IMAGE_NAME>.sif

## Execute custom stuff over the image:
singularity exec <IMAGE_NAME>.sif /bin/echo "Hello world!"

## Get an interactive shell:
singularity shell <IMAGE_NAME>.sif
```

We can bind additional directories (e.g. a shared dataset) to the container using the `-B` flag.
```bash
## Bind a directory. By default, the directory is mounted in the same location as in the host system:
singularity run/exec/shell -B <HOST/PATH/TO/DIR> <IMAGE_NAME>.sif

## We can also change this location with the `:` character:
singularity run/exec/shell -B <HOST/PATH/TO/DIR>:<SINGULARITY/PATH/TO/DIR> <IMAGE_NAME>.sif

## To specify various binds, we separate them with commas:
singularity run/exec/shell -B <DIR1>,<DIR2>,<DIR3:SINGDIR3> <IMAGE_NAME>.sif
```

### Running docker images
Singularity can actually pull Docker images and mount them as Singularity images:

```bash
singularity pull <IMAGE_NAME>.sif docker://<IMAGE_NAME>
```

---
## Building images
To build images we first need administrative access to a system with Singularity installed.
Keep in mind that this doesn't necessarily have to be the system where we'll run the image -- you can build it locally and then run it in your target HPC cluster.

Building an image requires us to write a **Singularity Definition File** (`.def` extension).
This is a text file with a series of statements that create a container image.
The ideal approach to do this is to store the definition file in the code repository alongside the application code.
This means that for every commit in the repository there is a definition file that can be used to reproduce a container with a known state.

A basic example definition file follows:
```
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'
```

### Bootstrap
The first two lines define where to bootstrap the image from.
This needs to be defined, since applications cannot run in a vacuum.
Note that in the example we're using a minimal Ubuntu 20.04 Linux Docker image:
```
Bootstrap: docker
From: ubuntu:20.04
```
* The `Bootstrap: docker` line is similar to prefixing an image path with `docker://`.
For a list of the available bootstrap agents, [check the docs](https://docs.sylabs.io/guides/3.5/user-guide/definition_files.html#preferred-bootstrap-agents).
* The `From: ubuntu:20.04` line says that we want to use the ubuntu image with the tag 20.04 from the Docker Hub.

### Post
```
%post
    apt-get -y update && apt-get install -y python3
```
In the post section we do tasks like package instalation, pulling data files, and undertaking local configuration.
These commands are standard shell commands, and are run within the context of the container image.
In the example, we're updating the package manager indexes and installing the `python3` package.
This definition file should run in an unattended, non-interactive environment, then we have to make sure that no interactive prompts appear on it.

### Runscript
```
%runscript
    python3 -c 'print("Hello World! Hello from our custom Singularity image!")'
```
This script is what's called when we run the image using the `run` command.

### Building
After setting up the `.def` file, we can build it using the `build` command:
```bash
sudo singularity build /home/singularity/<IMAGE_NAME>.sif /home/singularity/<IMAGE_NAME>.def
```

There are a lot of other stuff that we can add to the `.def` file, which you can see [in the docs](https://docs.sylabs.io/guides/3.5/user-guide/definition_files.html#sections).

---
## Neat details
### Directory binding
Singularity binds some directories from the host system into the container that we're starting.
*This is not copying files, but making existing directories available within the container.*
*Changes made in the container will persist in the host system.*

The mapping of directories onto the container is determined by the system administrator, so there might be differences across systems.
The default Singularity binding scheme is the following:
```
Host system:                                    Singularity container:
-------------                                   ----------------------
/                                               /
├── bin                                         ├── bin
├── etc                                         ├── etc
│   ├── ...                                     │   ├── ...
|   ├── group ──> user's group added to group ─────>├── group
|   |             file in container.            |   |
│   └── passwd ─> user info added to passwd   ─────>└── passwd
|                 file in container.            |
├── home                                        ├── usr
|   └── twig ───> user home directory made ─┐   ├── sbin
├── usr           available in container    |   ├── home
├── sbin          via bind mount.           └──────>└── twig
└── ...                                         └── ...
```

*You can add additional directories with the `-B` flag (see the Running an image section).

### User inheritance
Unlike docker (where everyone is the superuser), users from the host system are "inherited" to the Singularity image.
```bash
## Check user identity within image:
singularity exec <IMAGE_NAME>.sif /bin/whoami
```

This ensures that an user cannot access/modify/delete data from a container that they wouldn't be able to in the host system.

It's also worth noting that the filesystem itself is read-only, so even `root` wouldn't be able to edit anything.

### Cached imaged
Downloaded images are cached to avoid re-downloading them.

```bash
## Get a summary of cached images:
singularity cache list

## See the name and summary of each cached image:
singularity cache list -v

## Clean-up the whole cache:
singularity cache clean

# Fine-grained options can be seen with:
singularity cache clean --help
```

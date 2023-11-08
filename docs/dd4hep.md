# Geometry development with DD4hep
## Definition
After loading the `eic-shell`, we can load the `epic-nightly` build using
```bash
source /opt/detector/epic-nightly/setup.sh
```
*(or just source the convenient `/home/twig/.eic_bashrc` file).*

Then, we can look at the geometry resources, which are `.xml` files.
These files contain the parametrization of the entire detector, down to the subsystems.
The way the volumes are created from these definition files is defined in the `type` field, which points to the detector type plugin that interprets the parametrization.

The default entry point is `$DETECTOR_PATH/$DETECTOR_CONFIG.xml`, which links to the definition of all detector systems.
Each of these detector systems might link to additional detector subsystems, depending on their complexity.

We can look at, for example, the vertex barrel tracking detector via
```bash
cat $DETECTOR_PATH/compact/tracking/vertex_barrel.xml
```
```xml
<define>
  <constant name="VertexBarrelMod_length"   value="VertexBarrel_length"/>
  <constant name="VertexBarrelMod_rmin"     value="VertexBarrel_rmin"/>

  <constant name="SiVertexSensor_thickness" value="40*um"/>
  ...
</define>
```
where we'll see the detector's parameters are described in a `define` block, which can either use parameters previously defined in the included files, or which can be defined in the file itself.
For geometry development, best practice is to define detailed parameters in the subsystem end point file, but to refer to the central definitions in `$DETECTOR_PATH/compact/definitions.xml` for overal subsystem characteristics or interfaces with other subsystems.

For example, all dRICH geometry definitions can be found in the following file
```bash
cat $DETECTOR_PATH/compact/pid/drich.xml
```

## Visualization
All geometry included can be viewed with the ROOT geometry browser, but needs to be exported from the DD4hep format to the ROOT TGeo format.
To look at the geometry of the entire EIC, we run
```bash
dd_web_display $DETECTOR_PATH/$DETECTOR_CONFIG.xml
```
and then look at the generated display server ([http://127.0.0.1:8090](http://127.0.0.1:8090) by default).
Alternatively, we can add the `--export` flag so that the program exports a ROOT file that can be later viewed using ROOT.

If we want to look at the dRICH geometry, we run
```bash
dd_web_display $DETECTOR_PATH/epic_drich_only.xml
```

*The geometry view has to make decisions on what to draw to reduce the number of facets, so large detector systems may fail to be fully drawn.*

## Modifying geometry
First, to get a writeable version of the geometry software, we need to clone the repository to a writeable directory
```bash
git clone https://github.com/eic/epic
```
and build
```bash
$ cd epic
$ cmake -B build -S . -DCMAKE_INSTALL_PREFIX=install
$ cmake --build build -- install -j4
```

We see that the director `epic/install/share/epic` contains the same files as what we saw on `/opt/detector` inside the eic shell.
Finally, we want to set the path and related stuff to the local copy of the software
```bash
source install/setup.sh
```

If we need to make any modifications to the parametrization, the procedure is what should be expected:
* Create a branch/fork of the epic repository.
* Commit our changes to the local repository.
* Submit a pull request to include them in the main branch.

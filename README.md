# Generic KNIME Nodes
is a project aimed at the automatic generation of KNIME nodes for generic (command line) tools. It consists of functionality for KNIME developers (extension points, interfaces, ...), tool developers (automatic wrapping of tools into KNIME nodes based on the [Common Tool Description](https://github.com/WorkflowConversion/CTDSchema/tree/master); either by generating source code or recently automatically at KNIME startup, including an alternative plugin versioning approach) and KNIME users (intuitive configuration dialogs and node descriptions for any tool you can imagine plus additional nodes for the interaction with standard KNIME nodes).

## The GenericKnimeNodes KNIME plugin
It provides the source code for the KNIME plugin `com.genericworkflownodes.knime` providing basic functionality for further plugins depending on the Generic Workflow Nodes for KNIME mechanism. **NOTE:** For this functionality, at least KNIME 2.7.2 is required and you need to install 
the [KNIME File Handling Nodes](https://www.knime.com/file-handling). This functionality includes additional nodes for reading, writing, looping over and converting tables to and from files. For developers it provides extension points for:
   * registering conversion capabilities for new filetypes that show up in the table to file (and vice versa) nodes (so called mangler and demangler in the sources)
   * command line generation (to specify how to generate the command line from a tool description, if the standard way is not sufficient)
   * execution strategies (to specify how to schedule execution of the underlying tools; want to run it from inside a container? via ssh?)
   * versioned node set factories (to register new node generating plugins with a new version such that nodes shadowed by new versions show up as deprecated and are hidden in the node repository)

## The GenericKnimeNodes Plugin/Node generator
Via "ant" or exporting an executable jar file it provides a mechanism to create source code for your own complete
KNIME plugin:

1. Clone this repo
1. Switch to your GenericKnimeNodes directory
1. Call `ant`. This generates the required base plugin and runs the node generator. Further it will generate the source code for the KNIME plugin in a directory called ```generated_plugin/```
1. Load the GenericKnimeNodes folder and the generated plugin into the KNIME SDK (TODO Link) as Java projects.
1. Run the preconfigured KNIME Instance
1. Enjoy your new nodes!
1. If you want to build an update site for your new nodes, you can do that by creating an update site project
in Eclipse or use buckminster as described [here](https://www.knime.com/blog/creating-knime-update-sites-with-buckminster).

If you want to build your KNIME nodes from another directory
just call:
```
ant -Dplugin.dir=[your_path]
```
Do not forget to replace `[your_path]` with a valid directory.
e.g.
```
ant -Dplugin.dir=/home/jon.doe/my_plugin_sources
```
(without trailing slash!).

The following directory structure must be provided within
the specified directory:

```
/home/jon.doe/my_plugin_sources
  │
  ├── plugin.properties
  │
  ├── descriptors (place your ctd files and mime.types
  │                here)
  │
  ├── payload (place your binaries here)
  │
  ├── icons (the icons to be used must be here)
  │
  ├── contributing-plugins (optional; place all your OSGI
  │                         bundles and Eclipse plugin projects
  │                         here)
  ├── DESCRIPTION (A short description of the project)
  │
  ├── LICENSE (Licensing information of the project)
  │
  └── COPYRIGHT (Copyright information of the project)
```
  
See the sample* directories [here](https://github.com/genericworkflownodes/GenericKnimeNodes/tree/develop/com.genericworkflownodes.knime.node_generator/) for examples.

You can supply the executables for each node in the plugin.
The wrapped binaries have to be supplied in the payload directory.
Pleaser refer to [payload.README](payload.README) for further details.

If you want to ship additional custom nodes written as any other KNIME node in Java code,
you can create one or more OSGI bundles or Eclipse plugin projects (see KNIME node development documentation)
and copy them to a `contributing-plugins` directory.
The supplied plugins will be copied over and registered as part of the overall Eclipse feature (a collection of plugins)
that is generated and thus being installed together with the KNIME nodes themselves.

You have two options to make the target system (KNIME) execute your code:
1) You use the Activator/Plugin class (should be done automatically)
2) You extend the extension point "org.eclipse.ui.startup" if you need to do additional things during startup of your plugin (like checking
existance of another tool or library).

## What is generated?
An  installable feature that bundles the GKN base plugin as well as the plugin generated from your CTDs
plus any potential contributing plugins that you specified. This feature can then be added to an update site which can be shipped or hosted
and then installed under KNIME -> Install New Software...

## Versioning of your plugin/feature/update site
In general, Eclipse plugins have a version number following MAJOR.MINOR.PATCH.QUALIFIER.
For KNIME/Eclipse to recognize potential updates, there has to be an increase in the version number for your generated feature.
The main method of the raw nodegenerator.jar has three parameters: plugin-source-folder, build-output-folder and qualifier.
When run from ant, it automatically takes the last change date from your clone of this git repository as the third "qualifier" argument.
The third argument is used to represent the state of the GKN base plugin (if it changed after your sources changed,
there might have been changes in the source code of your Nodes and you would want an updated version).
For your plugin, it compares it to your qualifier in the plugin.properties file and updates it if it is alphabetically higher.
For the generated feature (comprising your plugin and contribs) it compares the third argument to *every qualifier* in your plugin versions (see your plugin.properties for your own plugin, as well as the MANIFEST.MF entry
BundleVersion of the contributing plugins) and takes the highest one for the resulting installable feature.

## Dynamic (Versioned) Node Generation
Recently, dynamic generation of nodes is supported. If you manually create a new plugin (dynamic plugins can not be automatically generated by the generator, yet) that provides a NodeSetFactory that extends the according extension points in this base plugin here, all descriptors (CTDs) are parsed automatically at KNIME startup and nodes are generated for them. More details [here](https://github.com/genericworkflownodes/GenericKnimeNodes/tree/develop/com.genericworkflownodes.knime/src/com/genericworkflownodes/knime/dynamic) and in the test [plugin](https://github.com/genericworkflownodes/de.openms.dynamic) that we provide for now.

## Update site building
1) Use Eclipse, manually write or generate (see dynamicPluginGeneration repo) an update site feature, features, plugins and platform-specific fragments
1) Get an rmap and a cquery (similar to the one in the buildresources repo) TODO extend
1) Get a p2 director executable (TODO add link)
1) Run
```
./director -r http://download.eclipse.org/tools/buckminster/headless-4.4 -r http://ftp-stud.fht- 
esslingen.de/pub/Mirrors/eclipse/releases/mars -r http://update.knime.org/build/3.1 -d 
/Users/pfeuffer/Downloads/director/bucky -p Buckminster \
-i org.knime.features.build.feature.group \
-i org.eclipse.buckminster.cmdline.product \
-i org.eclipse.buckminster.core.headless.feature.feature.group \
-i org.eclipse.buckminster.pde.headless.feature.feature.group \
-i org.eclipse.buckminster.git.headless.feature.feature.group
```
To install buckminster
1) Run buckminster on the cquery with TODO


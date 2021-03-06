# SpatialOS C++ Blank project

This is a SpatialOS project which can serve as a template for building
SpatialOS workers using the C++ SDK. It uses CMake as the build system. If
you're new to SpatialOS, have a look at [Introduction to the C++ worker SDK](https://docs.improbable.io/reference/latest/cppsdk/introduction).

## About managed and external workers

Have a look at the [Glossary entry for
Workers](https://docs.improbable.io/reference/latest/shared/glossary#worker) for a complete discussion and examples.

## Dependencies

This project requires [CMake](https://cmake.org/) to be installed. CMake is the build system used to build the project.

## Quick start

The build scripts for each project:

  1. Generate C++ code from schema
  2. Download the C++ worker SDK
  3. Create a `cmake_build` directory in the worker directory
  4. Invoke `cmake` with arguments depending on the target platform

To build and launch a local deployment execute the following commands:

```
spatial worker build
spatial local launch
```

To connect a new instance of the external C++ worker:

```
spatial local worker launch External local
```

## What the project does

It's a blank project so the short answer is - not much. The snapshot file in
this project contains a single entity with an ACL which matches the `Managed`
worker attributes. When you launch a deployment, a single instance of the
`Managed` worker will be started as configured in `default_launch.json`.

After connecting successfully both the `Managed` and `External` workers log a
message to SpatialOS which should be displayed in the output of `spatial local
launch` as it's running. Then they just spin in a loop processing the Ops list.

When SpatialOS disconnects a worker, a message is written to the console output
of the worker and it exits with an error status.

Instances of the `External` worker won't have any entities added to their view
because they don't have write access to anything in the snapshot.

## Project structure

The CMake project hierarchy doesn't exactly match the directory structure of
the project. For example the projects for workers add as subdirectories the
`schema` and `dependencies` projects.

This is how projects are structured in the directory:
```
+-- schema/CMakeLists.txt
+-- dependencies/CMakeLists.txt
+-- workers
    |-- External/
    |   |-- External/CMakeLists.txt
    |   |-- CMakeLists.txt
    |   |-- build.json
    |-- Managed/
        |-- Managed/CMakeLists.txt
        |-- CMakeLists.txt
        |-- build.json
```

This enables you to keep the worker directories free of CMake files for schema and dependencies while not needing a CMake file at the root of the project.

The `schema` directory contains a sample empty component called `blank`. It is
not used by the workers directly so feel free to delete it but it's there to
show how sources generated from the schema could be linked in the worker
binary. See `schema/CMakeLists.txt` which creates a library with all generated
sources.

You can see (and edit) the content of the snapshot in text format by running a command to convert it:

```
spatial project history snapshot convert --input=<path> --input-format=binary --output=<path> --output-format=text
```

### The worker project

The following applies to both the `Managed` and `External` worker projects but examples will only be about `Managed`.

After running `spatial build` the generated project by CMake will be in
`workers/Managed/cmake_build`. Exactly what it contains will depend on the
generator you use. A worker project includes 3 subdirectories in its
`CMakeLists.txt` - `dependencies`, `schema` and `Managed`. The first two are
not true subdirectories of `workers/Managed` in the file structure but their
binary directories are set as if they were.

## Attaching a debugger

If you use a Visual Studio generator with CMake, the generated solution contains several projects to match the build targets. You can start a worker from Visual Studio by setting the project matching the worker name as the startup project for the solution. It will try to connect to a local deployment by default. You can customize the connection parameters by navigating to `Properties > Configuration properties > Debugging` to set the command arguments. Using `receptionist localhost 7777 DebugWorker` as the command arguments for example will connect a new instance of the worker named `DebugWorker` via the receptionist to a local running deployment. You can do this for both worker types that come with this project. Make sure you are starting the project using a local debugger (e.g. Local Windows Debugger).

## Cloud deployment

As usual set the `project_name` field in `spatialos.json` to match your SpatialOS project name. Then upload and launch:

```
spatial cloud upload <assembly-name>
spatial cloud launch <assembly-name> default_launch.json <deployment-name> --snapshot=<snapshot-path>
```

See [`spatial cloud connect external`](https://docs.improbable.io/reference/latest/shared/spatial-cli/spatial-cloud-connect-external)
if you want to connect to a cloud deployment. In
addition, the `External` worker has a second external launch configuration
called `cloud` which could be used to connect provided that you know the
deployment name and have a login token:

```
spatial local worker launch External cloud <deployment-name> <login-token>
```

## Integrating with an existing project

If you have an existing project, to add a new C++ worker to it:

  1. Decide whether the worker you're adding will be used as a managed or
    external worker.
  2. Copy the corresponding directory (e.g. `workers/Managed`) into the workers
    directory of your existing project.
  3. In the worker project `CMakeLists.txt` set `SCHEMA_SOURCE_DIR` and
    `WORKER_SDK_DIR` to point to the CMake projects in your project that
    generate the corresponding targets and if the targets have different names
    from `Schema` and `WorkerSdk` also rename those.
  4. Add it to the `workers` definition in your SpatialOS launch configuration
    (e.g. `default_launch.json`)

## Windows release builds

On Windows, this project uses the [/MDd](https://msdn.microsoft.com/en-us/library/2kzt1wy3.aspx) build of the
C++ worker SDK. To build against the release /MD build of the C++ worker SDK, follow the instructions for
[obtaining the SDK](https://docs.improbable.io/reference/latest/cppsdk/setting-up#obtaining-the-sdk).

## Release notes

The following are versions of SpatialOS with the changes to this project
listed.

### 13.0.0

The only change in SpatialOS 13.0 is that SpatialOS no longer includes the Unity and Unreal SDKs.

### 12.0.0

- Use `worker::ComponentRegistry` where required.
- Set `is_connected` flag based on connection state.

### 11.0.1

Initial release.

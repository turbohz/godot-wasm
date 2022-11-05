# Godot Wasm

> **Warning**
> This project is still in its infancy. Not advisable to use in a release.

A Godot addon allowing for loading and interacting with [WebAssembly (Wasm)](https://webassembly.org) modules from GDScript. Note that this project is still in development.

This [GDNative](https://docs.godotengine.org/en/stable/tutorials/scripting/gdnative/what_is_gdnative.html) addon uses [Wasmer](https://wasmer.io) as the WebAssembly runtime.

## Installation

Installation in a Godot project involves simply downloading and installing a zip file from Godot's UI. Recompilation of the engine is *not* required.

1. Download the Godot Wasm addon zip file from the [releases page](https://github.com/ashtonmeuser/godot-wasm/releases).
1. In Godot's Asset Library tab, click Import and select the addon zip file. Follow prompts to complete installation of the addon.

## Usage

Once installed as an addon in a Godot project, the Godot Wasm addon class can be accessed via `Wasm`.

1. Create a Wasm module or use the [example module](https://github.com/ashtonmeuser/godot-wasm/blob/master/examples/wasm-consume/add.wasm).
1. Add the Wasm module to your Godot project.
1. In GDScript, instantiate the `Wasm` class via `var wasm = Wasm.new()`.
1. Load your Wasm module bytecode as follows replacing `YOUR_WASM_MODULE_PATH` with the path to your Wasm module e.g. *add.wasm*. The `Wasm.load()` method accepts a [PoolByteArray](https://docs.godotengine.org/en/stable/classes/class_poolbytearray.html).
    ```
    var file = File.new()
    file.open("res://YOUR_WASM_MODULE_PATH", File.READ)
    var buffer = file.get_buffer(file.get_len())
    wasm.load(buffer)
    file.close()
    ```
1. Define an array containing the arguments to be supplied to your exported Wasm module function via `var args = [1, 2]`. Ensure the number of arguments and argument types match those expected by the exported Wasm module function.
1. Call a function exported by your Wasm module via `wasm.fire("YOUR_FUNCTION_NAME", args)` replacing `YOUR_FUNCTION_NAME` with the name of the exported Wasm module function.

## Examples

[Examples](https://github.com/ashtonmeuser/godot-wasm/tree/master/examples) are provided for both creating and consuming/using a Wasm module.

### Wasm Consume (Godot)

A simple example of loading a Wasm module and calling its exported functions. The Wasm module used is that generated by the [Wasm Create example](#wasm-create-assemblyscript). All logic for this Godot project exists in `Main.gd`.

### Wasm Create (AssemblyScript)

An example [AssemblyScript](https://www.assemblyscript.org) project which creates a simple Wasm module with a single exported `add` function.

Note that WebAssembly is a large topic and thoroughly documenting the creation of Wasm modules is beyond the scope of this project. AssemblyScript is just one of [many ways](https://github.com/appcypher/awesome-wasm-langs#awesome-webassembly-languages-) to create a Wasm module.

## Developing

This section is to aid in developing the Godot Wasm addon; not to use the addon in a Godot project.

These instructions are tailored to UNIX machines.

1. Clone repo and submodules via `git clone --recurse-submodules https://github.com/ashtonmeuser/godot-wasm.git`.
1. Ensure the correct Godot submodule commit is checked out. Refer to relevant branch of the [godot-cpp project](https://github.com/godotengine/godot-cpp/tree/3.x) e.g. `3.x` to verify submodule hash. At the time of this writing, the hash for the godot-cpp submodule was `1049927a596402cd2e8215cd6624868929f5f18d`.
1. Download the Wasmer C library from [Wasmer releases](https://github.com/wasmerio/wasmer/releases) and add them to a *wasmer* directory at the root of this project. There should be *include* and *lib* subdirectories within the *wasmer* directory.
1. Install [SConstruct](https://scons.org/) via `pip install SCons`. SConstruct is what Godot uses to build Godot and generate C++ bindings. For convenience, we'll use the same tool to build the Godot TF Inference addon.
1. Compile the Godot C++ bindings. From within the *godot-cpp* directory, run `scons platform=PLATFORM generate_bindings=yes --jobs=$(sysctl -n hw.logicalcpu)` replacing `PLATFORM` with your relevant platform type e.g. `osx`, `linux`, `windows`, etc.
1. Compile the Godot Wasm addon. From the repository root directory, run `scons platform=PLATFORM` once again replacing `PLATFORM` with your platform. This will create the *addons/godot-wasm/bin/PLATFORM* directory where `PLATFORM` is your platform. You should see a dynamic library (*.dylib*, *.so*, *.dll*, etc.) created within this directory.
1. Copy the Wasmer dynamic libraries to the appropriate platform directory via `cp -RP wasmer/lib/. addons/godot-wasm/bin/PLATFORM/` replacing `PLATFORM` with your platform.
1. Zip the addons directory via `zip -FSr addons.zip addons`. This allows the addon to be conveniently distributed and imported into Godot. This zip file can be imported directly into Godot (see [Installation](https://github.com/ashtonmeuser/godot-wasm#installation)).

If frequently iterating on the addon using a Godot project, it may help to create a symlink from the Godot project to the compiled addon via `ln -s RELATIVE_PATH_TO_ADDONS addons` from the root directory of your Godot project.

## Known Issues

1. No WASI binding are provided to the Wasm module. This means that the guest Wasm module has no access to the host machines filesystem, etc. Pros for this are simplicity and increased security. Cons include not being able to generate truly random numbers (without a workaround) or run Wasm modules created in ways that require WASI bindings e.g. [TinyGo](https://tinygo.org/docs/guides/webassembly/) (see relevant [issue](https://github.com/tinygo-org/tinygo/issues/3068)).
1. No access to exported globals (see [roadmap](#Roadmap)).
1. No access to imported functions (see [roadmap](#Roadmap)).
1. No access to imported globals (see [roadmap](#Roadmap)).
1. Only `int` and `float` return values are supported. While workarounds could be used, this limitation is because the only [concrete types supported by Wasm](https://webassembly.github.io/spec/core/syntax/types.html#number-types) are integers and floating point.

## Relevant discussion

There have been numerous discussions around modding/sandboxing support for Godot. Some of those are included below.

- [Proposal](https://github.com/godotengine/godot-proposals/issues/5010): Implement a sandbox mode
- [Proposal](https://github.com/godotengine/godot-proposals/issues/4642): Add a method to disallow using all global classes in a particular GDScript
- [Pull Request](https://github.com/godotengine/godot/pull/61831): Added globals disabled feature to GDScript class

## Roadmap

Please feel free submit a PR or an [issue](https://github.com/ashtonmeuser/godot-wasm/issues).

- [x] Load module
- [x] Call exported function
- [x] Map export names to indices (allows calling by function name)
- [ ] Global exports
- [ ] Function imports
- [ ] Global imports
- [ ] Convenience methods to handle more data types e.g. `Array`, `Vector2`
- [ ] Optional WASI bindings (allows access to FS, etc.)

# Zig tracy client
Easy to use bindings for the tracy client C API.

Fork of https://github.com/neurocyte/zig-tracy (uptades to zig 0.13)
which is a fork of https://github.com/cipharius/zig-tracy.

## Version 0.11.2-dev (latest as of October 2024)
I had a painful experience in setting up Tracy. It requires exact
version match between server and client, and building the server on
Ubuntu 24.04 wasn't trivial:

- master builds with no issues, but you have to change a build flag to
enable X11
- last release 0.11.1 gives compiler errors on gcc 13 `writing 1 byte
into a region of size 0 overflows`
- 0.10 (which is the version of this client) had a different build
system, and the include paths to some font package are set up wrong
for Ubuntu 24.04
- 24.04 nix home-manager only had 0.9, and I think it's for Wayland. There
is an x11-package, but it's in not in the package source yet.
- Nixos unstable has 0.10, but it fails with cryptic errors. I've
modified `LD_LIBRARY_PATH` to get past one error, but am stuck at another.
Running under Wayland.

Hence forking this and updating the CLIENT to match a server I can
build. The client seems to build for any version without problems.


## Dependencies

* Zig 0.12.0-dev.3437+af0668d6c
* Tracy `(!)` **0.11.2** `(!)` (only for viewing the profiling session, this repository is only concerned with client matters)

## Features

* Designed to integrate well with build.zig
* Builds and statically links the tracy client - perfect for cross-compilation
* Uses Zig comptime to nullify the tracy markup when building with tracy disabled
* Provides Zig friendly bindings for:
    * Zone markup
    * Frame markup
    * Plotting
    * Message printing
    * Memory tracing via custom allocator

## Usage

See `./example` for how to set up `zig-tracy` with a Zig project.

In summary:
* Declare `zig-tracy` as a dependency in the `build.zig.zon`
* Configure `zig-tracy` dependency in `build.zig`
* Instrument the code using the provided Zig module
* Use Tracy UI/server to connect to the instrumented Zig application and explore the profiler data

## Building as a Shared Library

If your project needs to call tracy functions from multiple DLLs, then you need to build the tracy client as a shared library.

This is accomplished by passing the `shared` option, and (if you're using Windows) installing the resulting shared library next to your exe.

```zig
    const tracy = b.dependency("tracy", .{
        .target = target,
        .optimize = optimize,
        .shared = true,
    });

    const install_dir = std.Build.Step.InstallArtifact.Options.Dir{ .override = .{ .bin = {} } };
    const install_tracy = b.addInstallArtifact(tracy.artifact("tracy"), .{
        .dest_dir = install_dir,
        .pdb_dir = install_dir,
    });
    b.getInstallStep().dependOn(&install_tracy.step);
```

For additional context, see section 2.1.5 of the Tracy manual, "Setup for multi-DLL projects".

## Todo / Ideas

* Figure out why system sampling is broken
* Tracy fibers support, would make sense paired with Zig async
* GPU zone markup support
* Test callstack support

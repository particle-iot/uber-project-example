# Uber App Example

Repo showing example apps using libraries V2.

## What is sent to the compiler?

Here are three example apps, exactly as they would be sent to the compile server:

### [1. Initial app](/1-initial)

App structure created by `particle libraries init .`. This command will ask the user for things like app name, author etc. and propagate `library.propertes`.

**Libraries? Isn't this supposed to be an app?** All new apps are libraries too. They have the same directory structure, metadata files, tests.

### [2. After adding a library](/2-add-library)

Structure after issuing `particle libraries install neopixel@0.0.8`.
By default, installing library does:
* installs library in `~/.particle/libraries/:name/:version`
* adds `dependencies` line to `library.properties` file

This keeps apps small, you don't include dependencies in repository and when compiling, compiler has to make sure libraries are installed and in path.

Such library can be included like this:

```c
#include <library_name.h>
```

### [3. After vendoring library](/3-vendor-library)

Vendoring library is a term we use for installing library straight into app. This means it should be checked into repository along with app code.

`particle libraries install foo --vendor`

This doesn't add libraries to `dependencies` as library manager can only vendor libraries, not manage them. Vendored library will be saved in `lib/:name` directory.

This is useful when you want to:
* make sure all dependencies are bundled with app at specific version
* make changes to public library without publishing them

**Note: Particle library manager can't update vendored libraries.** It doesn't know if you modified the library since its installation so if you want to update it, you have to manually delete the library directory and vendor it again.

Vendored library can be included like this:

```c
#include <vendored_library_name.h>
```

As you can see global and vendored libraries share namespace. This is done by setting [`APPLIBS` env variable](https://github.com/particle-iot/firmware/pull/1009) with libraries paths.

## What does the compiler do with such apps?

1. First [`wiring-preprocessor` buildpack](https://github.com/particle-iot/buildpack-wiring-preprocessor) will transpile any `.ino` files into `.cpp` (this behaviour can be disabled with `#pragma PARTICLE_NO_PREPROCESSOR`).

2. Next [`install-dependencies` buildpack](https://github.com/particle-iot/buildpack-install-dependencies) will install all dependencies as vendored libraries:

	`particle libraries install --vendor`

	If a dependency is already vendored, it will be ignored.

3. Then the whole project including app code, dependencies and vendored libraries is passed to the [`particle-firmware` buildpack](https://hub.docker.com/r/particle/buildpack-particle-firmware/) which contains the compiler and firmware sources.

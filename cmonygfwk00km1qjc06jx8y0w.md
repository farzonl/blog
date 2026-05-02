---
title: "The "Warf" Way Part II: Tackling the macOS Frontier with osx-cross and Darling
"
seoTitle: "MacOS C++ Testing with osx-cross & Darling"
seoDescription: "Ditch expensive Mac hardware for your CI. Learn how to cross-compile and test macOS binaries directly on Linux using Docker, osx-cross, and Darling."
datePublished: 2026-05-02T06:23:02.279Z
cuid: cmonygfwk00km1qjc06jx8y0w
slug: the-warf-way-part-ii-tackling-the-macos-frontier-with-osx-cross-and-darling
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/88cd061a-b9c5-4262-adac-8a6787ae6b8a.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/f5eea7d9-4e2b-4d37-bce5-5333c955244e.png
tags: apple, linux, docker, darling

---

In my last post, [The Warf Way: Seamless Windows Testing](https://blog.farzon.org/2026/04/the-warf-way-seamless-windows-testing.html) we looked at how to streamline Windows testing by leveraging Docker and Wine. It provided a seamless way to validate Windows-specific logic without ever leaving the comfort of Linux. But as any cross-platform developer knows, Windows is only half the battle. The "Final Boss" of CI is almost always macOS.

Traditionally, testing for macOS meant maintaining expensive dedicated hardware or spinning up slow, proprietary VM instances in the cloud. Today, I'm excited to share a follow-up on how we are applying the "Warf Way" to Apple's ecosystem using [osx-cross](https://github.com/tpoechtrager/osxcross) for compilation and [Darling](https://www.darlinghq.org) for runtime testing. All packaged neatly inside [Dockerfile.osx-cross](https://github.com/farzonl/warfLang/blob/master/Dockerfile.osx-cross)

## The Challenge: Apple's Walled Garden

Compiling for macOS from Linux is notoriously difficult due to the requirement of the macOS SDK and specific linker behaviors. Testing that code is even harder, as macOS binaries require a Mach-O compatible kernel.

### WARF's Next Step: Declarative macOS Pipelines</h2>

The shift is subtle but important:
| Stage         | Old Model              | WARF Model                   |
| ------------- | ---------------------- | ---------------------------- |
| Compile       | Native macOS machine   | osxcross toolchain in Docker |
| Test          | Physical/virtual macOS | Darling runtime in Docker    |
| Orchestration | CI tied to hardware    | Fully containerized pipeline |


To solve this, we are using three powerhouse open-source projects:
- **osx-cross**: A toolchain that allows us to use [Clang](https://github.com/llvm/llvm-project/tree/main/clang)/[LLVM](https://llvm.org/) on Linux to target macOS.
- **Darling**: A translation layer (similar to Wine) that allows macOS binaries to run natively on Linux by implementing the Darwin subsystem.
- **macOS SDKs**: We need the SDK, headers, and frameworks Mac Apps expect.

### The Missing Link: Phracker's SDKs

The biggest hurdle to cross-compiling for macOS is the SDK. Legally and technically, you need the headers and frameworks found inside Xcode. Ordinarily, this involves downloading a 12GB Xcode image on a Mac and running an extraction script.

To make our Docker build truly portable and "headless", we rely on [Phracker MacOSX-SDKs](https://github.com/phracker/MacOSX-SDKs) (no longer maintained). This project provides pre-packaged, compressed versions of the macOS SDKs. By pulling a specific version (like 11.3 or 12.3) directly into our build context, we can automate the entire toolchain setup without manual intervention.

## The Solution: A Unified Dockerfile
### 1. Building the Toolchain

The <code>Dockerfile.osx-cross</code> starts by fetching build dependencies. Due to licensing, you'll need to package your own SDK, but once it's in the build context, <strong>osx-cross</strong> handles the heavy lifting:
    
```dockerfile
# Extract from Dockerfile.osx-cross
FROM ubuntu:22.04 AS build

# Install dependencies for osxcross
RUN apt-get update && apt-get install -y \
    clang llvm-dev libxml2-dev uuid-dev \
    libssl-dev bash patch make cmake xz-utils python3

# Set up the SDK and toolchain
COPY MacOSX11.3.sdk.tar.xz /osxcross/tarballs/
RUN UNATTENDED=1 ./build.sh  
```

### 2. Integrating Darling for Testing

While osx-cross gets us a binary, Darling acts as the bridge, providing the Mach-O loader and basic Apple frameworks needed to execute command-line tools.
    
```dockerfile
# Setup Darling for runtime tests
RUN apt-get install -y darling-dkms darling
ENTRYPOINT ["darling", "shell"]  
```
    
> Verdict: Darling is incredible for CLI tools, though your mileage may vary if you need to test UI frameworks.


## Why This Matters

- **No device context switch**: Compile and test for Linux, Windows (via Wine), and macOS (via Darling) on a single machine.
- **Reduced Latency**: No waiting for a Mac mini in a server rack. Tests run as fast as your Linux runner allows.
- **Version Parity**: Every developer uses the exact same SDK version and toolchain headers defined in the Dockerfile.

## Conclusion

The industry has spent years treating platforms as places you log into or wait on. But platforms were always contracts. A set of assumptions about binaries, syscalls, and behavior. When composed correctly, those contracts can be reconstructed and containerized. If a platform can be modeled, it can be versioned. If it can be versioned, it can be reproduced. And if it can be reproduced, it no longer owns your pipeline.

> First published 4/21/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/the-warf-way-part-ii-tackling-macos.html)
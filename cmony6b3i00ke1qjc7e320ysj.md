---
title: "The "Warf" Way: Seamless Windows Testing on Linux using MinGW & Wine"
seoTitle: "Cross-Compiling C++ with Docker, MinGW & Wine CI"
seoDescription: "How warfLang uses Docker, MinGW-w64, and Wine to build and test a cross-platform C++ parser with reliable Windows CI via GitHub Actions."
datePublished: 2026-05-02T06:15:09.489Z
cuid: cmony6b3i00ke1qjc7e320ysj
slug: the-warf-way-seamless-windows-testing-on-linux-using-mingw-wine
canonical: https://blog.farzon.org/2026/04/the-warf-way-seamless-windows-testing.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/80f01020-4a7e-4b30-9246-187806808d47.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/949d768e-7395-4467-89b6-0b68fcc8e69a.png
tags: linux, docker, mingw, cross-compile

---

In the world of language development, even a "toy" language needs a solid foundation. My project, **[warfLang](https://github.com/farzonl/warfLang)**, isn't trying to replace C++ or Rust. It’s an LL(1) parser written in C++ that generates an Abstract Syntax Tree (AST)—essentially a glorified calculator that supports variable definitions.

![warflang logo](https://blogger.googleusercontent.com/img/a/AVvXsEg2ytcGHR0oYqJtDJvUAxCA6z575HNFLeCsj1Eut6Bbtd0WJhFwg3m45gOc4GjBBP4kYfZJa-vsPnK32cW1gW6GooaojiOym7CWshDD5SLKcyB0objMDWsPUL7c8NG6gX1HsjIz2kD8eJC-ZIEAO8AvVRkXHpKfEU_QtjCywkM5BA6b0x8lIDc1ATMYskI)
  
  

But even a calculator needs to count correctly on every platform. To ensure **warfLang** remains truly cross-platform, I needed a CI/CD pipeline that could verify its logic on Windows without the overhead of actually owning a Windows machine.

## The Problem: Cross-Platform Logic is Tricky

Even for a high-level parser, moving from Linux to Windows introduces subtle behavioral risks and differences especially when you are compiling acros different compilers like CL, Clang, and GCC or different libC implementations. 

Inspired by [**Gianni Milanesi’s** post](https://www.metricpanda.com/rival-fortress-update-11-cross-compiling-for-three-platforms/), I realized I didn't need a native Windows environment to find out. I just needed a containerized "Build Appliance." 

> _Note this post will not be the promised part 5 of the Cross-Compiling series since we are using **mingw-w64**  and not clang_.

## The "Warf" Solution: Dockerizing the Toolchain

Instead of manually managing cross-compilers, I use a custom Dockerfile [Dockerfile.mingw-cross](https://github.com/farzonl/warfLang/blob/master/Dockerfile.mingw-cross). This encapsulates the entire MinGW-w64 suite, ensuring that the environment where I build **warfLang** today is the exact same one I'll use six months from now.

### Why Docker makes it easy: 

1. **Zero Setup:** A new contributor doesn't need to install `binutils-mingw-w64`. They just run the container.

2. **Consistency:** It's a container, it always the works the same thats the point.

3. **The "Magic" Compiler**: We get instant access to `x86_64-w64-mingw32-gcc`, which treats our C++ code as a native Windows project.

```dockerfile
FROM ubuntu:latest
  
# Install MinGW for building and Wine for testing
RUN apt-get update && apt-get install -y \
 mingw-w64 \
 cmake \
 wine \
 && rm -rf /var/lib/apt/lists/\*

...

RUN cmake -S . -B build \
 -GNinja -DCMAKE\_BUILD\_TYPE=Release \
 -DCMAKE\_C\_COMPILER=x86\_64-w64-mingw32-gcc-posix \
 -DCMAKE\_CXX\_COMPILER=x86\_64-w64-mingw32-g++-posix \
 -DBuildTest=TRUE \
 -DCMAKE\_SYSTEM\_NAME=Windows -DCMAKE\_CROSS\_COMPILING=TRUE
```

## Bundling Wine: Testing the Calculator

The real value of the "Warf" way is the inclusion of **Wine**. For a project like **warfLang** , testing is straightforward: I feed the parser an expression (like `x = 5; x * 2;`), and I expect a specific AST output. By bundling Wine in the Docker image, I can run the Windows version immediately after compiling it without having to context switch to a new device. This works the same on my local dev machines as well as in the CI.

### Why this is vital for a "Toy" Language:

- **It isn't**: but it is fun 😂!
- **Headless Validation**: The real reason I did this no context switching for testing cross plat. Using `wine ./warf_tests.exe` in a headless Docker container gives me instant feedback if something works on windows without having to have a dedicated windows device.

## Leveraging GitHub Actions

By using this crossPlat builder approach, my GitHub Actions workflow is faster and [cheaper (see runner pricing)](https://docs.github.com/en/billing/reference/actions-runner-pricing). I pull a single Linux-based image, build the Windows `.exe file`, and run the test suite through Wine.

![Runner prices](https://blogger.googleusercontent.com/img/a/AVvXsEg49EpJab7EvdSIzht5lZj_3EkKnEaaOQXk-bYiHPQvL5vBvPLB86qpFbyAiEyfa7EBGHZuTUHRrWmatoEKQpRERwDYm2EDRlZXqaDW7tQyhSgxqJ4nT6yKQ9nkgCIfD7HytR9gsRmONj7GlFyvL0IxHBULW0OXzuPN9DxJ_ljIRDmG44kRZQpEcC0Q2Jo)
   
When I see the "Green Checkmark" now it proves that the "calculator" works on Windows, even though it was born and raised on Linux.

## Conclusion

Whether you're building a massive compiler or a glorified calculator like **warfLang** , the "Warf" way combining Docker, MinGW, and Wine provides a professional, reproducible, and verifiable way to ship to Windows users without ever leaving the comfort of Linux.

_Check out the implementation and play with the parser in the [WarfLang WASM demo](https://farzon.org/warfLang/)._

> First published 4/19/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/the-warf-way-seamless-windows-testing.html)
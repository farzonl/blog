---
title: "Standardizing HLSL: Bringing the Clang Era to Ecma"
seoTitle: "HLSL joins Ecma"
seoDescription: "HLSL moves toward a formal, vendor-neutral standard under Ecma TC57—boosting portability, consistency, and cross-platform shader development."
datePublished: 2026-05-01T23:49:04.389Z
cuid: cmonkdssj00fj1qjc5mrth6u1
slug: standardizing-hlsl-bringing-the-clang-era-to-ecma
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/068b9af5-2c5e-4b42-9516-9c432ef01e5e.webp
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/f9090e7f-5447-466f-a768-302a635e3905.webp
tags: vulkan, spir-v, ecma, clang, hlsl, directx, tc-57, llvm-project

---

> Originally published at [blog.farzon.org](https://blog.farzon.org/2026/04/standardizing-hlsl-bringing-clang-era.html) on April 6th

For a long time, HLSL has lived in a bit of a "trust me, it works" state. It’s the language that powers basically every high-end frame you see on a screen, but for most of its life, it lacked a formal, vendor-neutral definition. Hell even among single vendor the three implementations (FXC, DXC, & Clang-DXC) don't always agree. So what are we going to do about this?

I’m excited to share that I’ve taken on the role of **Vice-Chair for the new Ecma Technical Committee 57 (TC57)**. Our mission? To finally turn HLSL into a formal industry standard.


### Why a Standard? Why Now?

The "why" is pretty simple: portability and predictability. We are currently in the **Clang era** of HLSL. [DXC](https://github.com/microsoft/DirectXShaderCompiler) and [github/hlsl-specs](https://github.com/microsoft/hlsl-specs) was our first dip of our toes into open source. We have the muscle memory now to be able to work in the open and have built great partnerships with hardware vendors, game engine developers, and Vulkan platform contributors. We now know by upstreaming HLSL into Clang/LLVM, we can make the final steps necessary to moving away from proprietary ownership and toward a compiler infrastructure that the entire industry can contribute to and rely on. But a compiler implementation isn't a substitute for a spec thats why C\\C++ have ISO.

I can't stress enough the portability reason. Thats one of the reasons we annouced 2 years ago our long term plans to retire DXIL and move to [SPIR-V for shader model 7](https://devblogs.microsoft.com/directx/directx-adopting-spir-v/) for DirectX.

Standardizing through \*\*Ecma International (\*\*these are the folks behind JavaScript and C#) gives us a neutral ground. It means HLSL isn't just a "Microsoft or Windows or DirectX thing" anymore; it’s a first-class language for the whole graphics ecosystem. To be fair HLSL hasn't been a DirectX only thing for a while both DXC and Clang-DXC have evolving and in some cases robust support for Vulkan\\SPIR-V.

### The "State of the Union"

If you want the official annoucment, you can check it out here: [Microsoft announcement on the DirectX blog](https://devblogs.microsoft.com/directx/standardizing-hlsl/). It covers the high-level goals and the move toward a royalty-free, open standard.

However, if you want a more "off the cuff" look at the technical reality—the technical debt we're clearing, and where we're headed with the standard you should absolutely read [Chris’s blog post on the State of the HLSL Union](https://www.abolishcrlf.org/2026/02/10/HLSLState.html). It’s a great deep dive into what this actually means for the people writing the shaders.

### Building in the Open

You can check out our charter and the technical committee details on the [official Ecma TC57 page](https://ecma-international.org/technical-committees/tc57/), or dive straight into the code and meeting notes at the [HLSL-TC57 GitHub org](https://github.com/hlsl-tc57).

We’re focusing on making the language more rigorous, improving the "Offload Test Suite" to ensure things actually work across different backends (DXIL, SPIR-V, etc.).

It's a huge step for graphics engineering. We're finally treating the shader language with the same systems-level rigor as C++ or Rust.

* * *

### Quick Links & Resources:

*   **Official Committee:** [Ecma International TC57](https://ecma-international.org/technical-committees/tc57/)
    
*   **Development Hub:** [HLSL-TC57 on GitHub](https://github.com/hlsl-tc57)
    
*   **The Microsoft Announcement:** [Standardizing HLSL (DirectX Blog)](https://devblogs.microsoft.com/directx/standardizing-hlsl/)
    
*   **Community Deep Dive:** [The State of HLSL (Abolish CRLF)](https://www.abolishcrlf.org/2026/02/10/HLSLState.html)
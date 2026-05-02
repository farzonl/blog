---
title: "Bringing HLSL Support to Clangd: The HLSL teams  first GSoC project"
seoTitle: "GSoC 2026: Bringing HLSL Support to Clangd"
seoDescription: "HLSL's first GSoC project aims to add Clangd support improving IDE features like autocomplete & navigation for shader devs in graphics workflows."
datePublished: 2026-05-02T03:30:33.633Z
cuid: cmonsamu700gr1qjc1akz6hif
slug: bringing-hlsl-support-to-clangd-the-hlsl-teams-first-gsoc-project
canonical: https://blog.farzon.org/2026/04/bringing-hlsl-support-to-clangd-hlsl.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/880d1e3f-be7f-465a-aee5-d152b79c266a.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/8e6af33d-8af6-4a77-a890-f9c97c5f26c4.png
tags: hlsl, gsoc2026, llvm-project

---

I am thrilled my team is sponsoring our first of many featured project for the **2026 Google Summer of Code** [Enable Clangd support for HLSL](https://llvm.org/OpenProjects.html#enable-clangd-hlsl-support).

If you are a student developer interested in compilers, developer tooling, or graphics programming, this is a unique opportunity to contribute to the LLVM ecosystem and improve the lives of thousands of shader developers.

### What is HLSL and Clangd?

**[HLSL](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl)** (High-Level Shader Language) is the primary language used for GPU shader programming in [DirectX](https://devblogs.microsoft.com/directx/landing-page/) and [Vulkan](https://www.vulkan.org/) via [SPIR-V](https://www.khronos.org/spirv/) environments. Currently, there is a massive effort underway to bring first-class HLSL support directly into Clang.

**[Clangd](https://clangd.llvm.org/)** is the language server ([LSP](https://en.wikipedia.org/wiki/Language_Server_Protocol)) that provides IDE features like code completion, go-to-definition, and refactoring for C++. Since HLSL is built on the C++11 standard, clangd already "mostly" works, but it struggles with HLSL-specific constructs. This project aims to close that gap.

### The Project Goal

The objective is to move from "semi-functional" to "holistic" support. This isn't just about writing code; it’s about understanding the nuances of a domain-specific language and designing how tools should interact with it.

### The Roadmap
1. **Survey**: Identify exactly where clangd breaks when handling HLSL.
2. **Architect**: Write an RFC (Request for Comments) to propose how clangd should handle these unique language constructs.
3. **Implement**: Begin the groundwork and implementation to fix these gaps.

### Project Details
- **Size**: Medium (~180 hours)
- **Difficulty**: Medium
- **Mentors**: My teammates Finn Plummer and Ashley Coleman
- **Required Skills**: Intermediate C++, an understanding of LSPs, and a hunger to learn the HLSL specification.

### Why apply?

By taking on this project, you’ll be mentored by two bright engineers who are deeply involved in the LLVM and HLSL space. You will gain hands-on experience with the LLVM codebase which is one of the most influential open-source projects in the world and your work will directly improve the developer experience for gpu compilers.

### How to Get Involved

If you’re interested in applying, we encourage you to join the conversation early!

- [**Official LLVM Project Listing**](https://llvm.org/OpenProjects.html#enable-clangd-hlsl-support)
- [**Discussion Thread**](https://discourse.llvm.org/t/gsoc-2026-enable-clangd-support-for-hlsl/89664)

Feel free to reach out on the LLVM Discourse or Discord to introduce yourself and ask questions. We can’t wait to see what you’ll build!

> First published 4/17/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/bringing-hlsl-support-to-clangd-hlsl.html)
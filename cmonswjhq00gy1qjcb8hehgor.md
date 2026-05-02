---
title: "Automating C++ Code Coverage with LLVM and GitHub Actions"
seoTitle: "Automate C++ Code Coverage with LLVM & GitHub Actions"
seoDescription: "Learn to automate C++ coverage using  LLVM-Cov, and GitHub Actions to generate reports and publish them via GitHub Pages CI/CD."
datePublished: 2026-05-02T03:47:35.729Z
cuid: cmonswjhq00gy1qjcb8hehgor
slug: automating-c-code-coverage-with-llvm-and-github-actions
canonical: https://blog.farzon.org/2026/04/automating-c-code-coverage-with-llvm.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/f9b9ae24-e790-4280-98d7-7bbcfd54d344.jpg
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/b9fc6fb6-aa0b-487f-a749-74f23cd9d64b.jpg
tags: code-coverage, github-actions, clang, llvm-project

---


Setting up an automated coverage report is a powerful way to visualize the effectiveness of your test suite. This post breaks down how the [warfLang repository](https://github.com/farzonl/warfLang) generates and publishes its reports, moving from raw source code to a live site hosted on GitHub Pages.

We will look at how this is done using Clang and LLVM-Cov, which provides highly accurate source-based coverage.

![Code Coverage results of WarfLang](https://blogger.googleusercontent.com/img/a/AVvXsEh3UVv9cxst3ryptGXV2eDcrudftUH2zuJCaKUn7aABrpuKHm1xChDx5Y9JXPzsaoXtdcDIxfVzlIjlAf41AZgChxXxNfAJsd_EK8S0abcbpqv8WoVzP-JsGINvfhWf9sQQmmeTdZXjJduC_glNNaE7_1SlIwfCpWnA4FYjLTWrFrYhPRz10ds27r5KHyE)
  
### The Foundation: Clang and LLVM-Cov

To understand the automation, we first need to look at what is happening under the hood. Unlike GCC, which traditionally uses `gcov`, Clang utilizes "Source-based Code Coverage." This method is generally more precise because it maps coverage directly to the [Abstract Syntax Tree (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree).

To enable this, we use two critical compiler flags:

- **`-fprofile-instr-generate`**: Tells the compiler to insert counters into the code to track how many times each block is executed.
- **`-fcoverage-mapping`**: Adds metadata to the binary that associates these counters with specific source code locations.

> _(Note: If you are migrating to Clang or want to understand how these flags interact with tools like CTest, [Kaspar Giger’s guide on Clang Coverage](https://www.kgmw.ch/posts/clang-coverage/) is an excellent primer)._

### Step 1: Configuring the Build in GitHub Actions

In the [`coverage-report.yml` workflow](https://github.com/farzonl/warfLang/blob/master/.github/workflows/coverage-report.yml), the first step is configuring CMake to use Clang and apply these flags.

```cmake
cmake $GITHUB_WORKSPACE -DCMAKE_BUILD_TYPE=$BUILD_TYPE \
  -DCMAKE_CXX_FLAGS="-fprofile-instr-generate -fcoverage-mapping" \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++
```

It is crucial to remember that the linker also needs the `-fprofile-instr-generate` flag to ensure the final executable is capable of writing the coverage data at runtime.

### Step 2: Generating `.profraw` Files

When you run a binary compiled with these flags, it doesn't output a readable report immediately. Instead, it produces a `.profraw` (Profile Raw) file. This is a highly efficient, compact binary format that contains the raw counter data.

The `warfLang` workflow uses the `LLVM_PROFILE_FILE` environment variable to explicitly name the output file so it isn't overwritten:

```shell
LLVM_PROFILE_FILE="warfLang.profraw" ./build/test/warfLang_TEST
```

If you are curious about what exactly is happening inside these binary files, [Leo Di Donato wrote a fascinating deep-dive demystifying the `.profraw` format](https://leodido.dev/demystifying-profraw/), explaining how the headers, function hashes, and execution counters are structured.

### Step 3: Merging into `.profdata`

Because a project might involve multiple test runs (e.g., unit tests, CLI tests, integration tests), you often end up with multiple `.profraw` files. To create a unified report, you must merge them using `llvm-profdata`.

```shell
llvm-profdata merge -sparse warfLang*.profraw -o warfLang.profdata
```

The `-sparse` flag is a useful optimization that reduces the size of the output `.profdata` file by omitting empty or zero-count blocks.

### Step 4: Creating the HTML Report

Once we have the merged `.profdata`, we use `llvm-cov show` to generate the actual visual report. This step requires the original binary because the mapping between the counters and the source code is still stored within that executable.

```shell
llvm-cov show -output-dir=build/out/report -format=html \
  -instr-profile=warfLang.profdata \
  -object=build/test/warfLang_TEST \
  build/src/cli/warfLang src
```

- **`-object`** : Specifies the binary files to look in for coverage mapping.
- **`-format=html`** : Generates an interactive website.
- **`src`** : Tells the tool which directory contains the source code to be mapped.

### Step 5: Publishing to GitHub Pages

The final piece of the puzzle is making the `build/out/report` directory accessible online. The `warfLang` workflow achieves this by initializing a new git repository inside that directory, committing the HTML files, and force-pushing it to a dedicated branch (`gh-pages-coverage`).

```yaml
- name: Force push to destination branch
  uses: ad-m/github-push-action@master
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    branch: gh-pages-coverage
    directory: ${{github.workspace}}/build/out/report
```

By pointing GitHub Pages to the `gh-pages-coverage` branch, the report becomes live at [**farzon.org/warfLang/coverage/**](https://farzon.org/warfLang/coverage/). This creates a seamless CI/CD loop: every push to the master branch triggers a build, runs tests, generates a new report, and updates the public-facing coverage site automatically.

* * *

### Resources for Deeper Learning

If you want to explore the underlying tools and mechanics further, these articles provide excellent context:

- **[Code Coverage with Clang](https://www.kgmw.ch/posts/clang-coverage/) by Kaspar Giger**: A great overview of the Clang compiler flags and how to integrate them cleanly using CMake.
- **[Demystifying the profraw format](https://leodido.dev/demystifying-profraw/) by Leo Di Donato**: A technical deep-dive into the LLVM source code to explain exactly what is written into a `.profraw` binary file.
- **[Code coverage with llvm-cov](https://medium.com/@sag50692/code-coverage-with-llvm-cov-16639b71f6a5) by Anastasiia**: A practical, step-by-step tutorial on local coverage generation, merging, and setting up CMake for GoogleTest.

> First published 4/17/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/automating-c-code-coverage-with-llvm.html)
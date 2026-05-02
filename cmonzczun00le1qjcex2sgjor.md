---
title: "A Retrospective on Fuzzing Edge for Android: Looking beyond the Emulator"
seoTitle: "Reducing Costs & Noise with Android Fuzzing on Azure"
seoDescription: "Former MS Edge security lead shares how infra constraints led to a novel Android-x86 fuzzing plan that cut Azure costs & eliminated emulator noise."
datePublished: 2026-05-02T06:48:21.121Z
cuid: cmonzczun00le1qjcex2sgjor
slug: a-retrospective-on-fuzzing-edge-for-android-looking-beyond-the-emulator
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/9087e09f-a2d6-4a10-b6ba-242ab4914bb8.png
tags: azure, fuzzing, edge-browser, androidx86

---

When I was the head of Microsoft's Edge security for the US market, we ran into an infrastructure challenge that forced us to completely rethink how we fuzzed Android.

The project we built no longer exists at Microsoft, and the [Android-x86 project](https://www.android-x86.org/) we relied on is now officially dead and unsupported. However, the architectural pivot we made to get around cloud compute constraints remains a novel approach to Android fuzzing—one that solved a myriad of technical headaches while drastically cutting costs.

Here is a look back at how we bypassed the Android Emulator entirely to find better, real-world bugs.

## The Catalyst: Losing Nested Virtualization

Fuzzing has always been a core focus of the Edge security posture. Using our own tooling alongside open-source frameworks, we achieved millions of fuzz hours per month across desktop environments. But doing this at scale for Android always presented unique friction.

In 2022, that friction hit a boiling point. Due to cost-cutting measures, our fuzzing budgets were reduced. We had to migrate off of public Azure to the Microsoft 365 substrate for both security and cost reasons. The unintended side effect of this migration was a severe reduction in our quota for CPU cores that supported **nested virtualization**.

 ![The Android Emulator works best with nested virtualization.](https://i.imgur.com/iQRZp5c.png)

Without nested virtualization, running the standard Android Emulator in the cloud was impossible. We needed a workaround.

## Why the Android Emulator Was Failing Us Anyway

Losing the ability to run the emulator initially felt like a massive blow, but in retrospect, the emulator was already polluting our fuzzing efforts. When we fuzzed on the emulator, we spent a massive amount of time triaging crashes that had nothing to do with Edge.

The emulator suffered from several fatal flaws for our use case:

- **QEMU-Specific Code Paths**: Many bugs we found hit `qemu`-specific emulation paths that a real-world mobile user would never actually experience.
 ![Emulator bug not browser.](https://i.imgur.com/eHcSgKe.png)
- **Driver Crashes**: The emulator frequently hit `nvoglv64.dll`, an Nvidia driver bug that would simply crash the emulator outright.
 ![nvoglv64.dll driver bugs](https://i.imgur.com/pQgS4K7.png)
- **The Swiftshader Timesink**: The only way to stop the Nvidia crashes was to enable Swiftshader for the emulator. However, Google had already deprecated Swiftshader, meaning any bugs we found there were a complete waste of time to report or investigate.

We needed a solution that tested real Android code paths, avoided emulation artifact bugs, and didn't require nested virtualization SKUs.

## The Solution: Direct-to-Azure Android-x86 VHDX Images

To solve the compute constraint, we turned to the Android-x86 project. Instead of emulating Android hardware, we built Android-x86 `.vhdx` images and uploaded them directly to Azure to run as standalone VMs. (For a look at the mechanics of this, there is a great breakdown on [running Android inside Azure here](https://mahsa-hanifi.medium.com/running-android-inside-azure-68977c687ff5)).

 ![A run of AndroidX86 in Azure](https://i.imgur.com/C7W9MR8.png)

Because we were running standard VMs instead of nested virtualization SKUs, our compute costs plummeted. We could suddenly run **two Android-x86 VMs for every one Android Emulator VM** we previously ran.

In the short term, the development time required to create the images and establish communication protocols was slightly higher than just spinning up an emulator. But the image creation and communication processes ended up being a massive success. The long-term Azure cost reductions eventually set a model for the rest of Edge.

## Decoupling the Architecture

Running standalone Android-x86 instances meant we could no longer run our test case generators, HTTP servers, and browser automation on the same VM as the target—something we took for granted on Windows and Linux.

To adapt, we decoupled the architecture. We built a new producer-consumer model where a robust Linux server handled the heavy lifting of generating test cases. The Android-x86 VMs acted purely as headless consumers, utilizing Mozilla's [Grizzly framework](https://github.com/MozillaSecurity/grizzly/tree/android-rebase) to handle the browser automation.

Use of android adb tools worked really smoothly after all of this

 ![Android adb with Androidx86 vm on Hyper-V](https://i.imgur.com/fdq3dtT.png)

By pushing our ASAN (AddressSanitizer) builds of the full Edge browser to these VMs, we maximized our chances of finding both security and reliability issues.

## The Takeaway

Even though Android-x86 was an older version of Android than what was available via the emulator, the net result was surprising: we found more _real-world_ bugs in the browser.

 ![Bugs found on Androidx86](https://i.imgur.com/moYugL9.jpeg)

I even found one that someone beat me to fixing.

 ![Bug I had a fix for](https://i.imgur.com/mHNGHqS.jpeg)

By being forced off of nested virtualization, we accidentally cured our emulator-noise problem. The project may be sunsetted today, but it stands as a great example of how strict budget and infrastructure constraints can force security teams to build leaner, smarter, and ultimately more effective testing environments.

> First published 4/25/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/a-retrospective-on-fuzzing-edge-for.html)
---
title: "MemGC: A Retrospective on a Bygone Frontline of Browser Security"
seoTitle: "MemGC Explained: Killing Use-After-Frees in Spartan Edge"
seoDescription: "How MemGC used concurrent mark-sweep GC to mitigate use-after-free bugs in browsers. Plus its limits, blind spots, and lessons for modern systems."
datePublished: 2026-05-02T05:05:10.281Z
cuid: cmonvoayu00if1qjc6sue1uor
slug: memgc-a-retrospective-on-a-bygone-frontline-of-browser-security
canonical: https://blog.farzon.org/2026/04/memgc-retrospective-on-bygone-frontline.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/f0db731d-88ac-4cf5-888c-b63d4c50a548.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/ce98ae3a-678d-405c-a8e5-ec5b8100b783.png
tags: security, chakra, browser-security, browser-internals, memgc, spartan-edge

---

Looking back at my time on the Chakra and the OG Edge team, few projects I had the opportunity to work on feel as consequential to my development as an engineer as my opportunity to work on **[MemGC (Memory Garbage Collection)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/overview-of-threat-mitigations-in-windows-10)**. In the 2014-16 time period the largest amount of security bugs filed against Internet Explorer were [Use-After-Frees (UAFs)](https://chs.us/2026/03/use-after-free-understanding-a-classic-memory-corruption-bug/).

![UaF infographic](https://blogger.googleusercontent.com/img/a/AVvXsEieXh5Zzzrn79okLg410mTkJIxN7MHFxQwbu2O7EaW0nWzDMayh4RJ9mHbj6yqF78hTr7W-D9_7ix9SyXcVm_NNNMSUd3E4trxQBAFrMj5GwTqkC0yQnT362pmHA6YnLsOlmMGElcTqG5pbHHqvW87KqAgCAQv78C6stgnNFdGTgjewhxTiWfCxoHhk2uk)
  
MemGC was our architectural response: a garbage collector designed not just for performance, but as a hard security boundary. It was good enough that it recieved praise from [Google project zero](https://projectzero.google/2017/09/the-great-dom-fuzz-off-of-2017.html): "_MemGC is an example of a useful mitigation that results in a clear positive real-world impact"_. I do love quoting this 😂.

## The Core Idea: Turning UAF into a GC Problem

Before MemGC, we tried mitigations like _[Isolated Heap](https://web.archive.org/web/20150524172425/http://labs.bromium.com/2015/01/17/use-after-free-new-protections-and-how-to-defeat-them/)_ and _[Delay Free](https://www.securityweek.com/microsofts-use-after-free-mitigations-can-be-bypassed-researcher/)_. They were clever "band-aids" that made exploitation harder but didn't solve the root cause. If a developer forgot to null a pointer after a `free()`, the door remained open.

MemGC changed the game by bringing the **[Concurrent Mark-Sweep (CMS)](https://en.wikipedia.org/wiki/Concurrent_mark_sweep_collector)** algorithm from the JavaScript engine (Chakra) into the heart of the DOM. We essentially built a safety net:

- **Reference-based Lifetime**: Instead of manual `free()` calls, objects were only reclaimed when the GC could prove no references to them remained.
- **The Conservative Scan**: We treated the stack and registers conservatively. If a value looked like a pointer to a MemGC object, we kept that object alive. This effectively killed "immediate" UAF exploits.

## What it Was Good At: Structural Defenses

From a developer’s perspective, MemGC was incredibly successful at **"Vulnerability Class Elimination."**

1. **Automation of Safety**: It removed the cognitive load from DOM developers. The GC handled the cleanup of complex DOM nodes automatically.
2. **Performance Balance**: By using a concurrent mark-sweep, we kept the UI thread responsive while the GC did the heavy lifting on a background thread.
3. **Heap Integrity**: The data structures we used, like the `HeapBlock32Map`, allowed us to quickly determine if an address was a valid object start, preventing many "middle-of-object" pointer tricks.

## Where it Failed: The "Blind Spots"

No mitigation is perfect. As developers, we had to confront the reality that hackers are just as creative as we are. One of the most interesting "failures" was a limitation in visibility between different engine components.

As detailed in the _"Seeing Double"_ research, we discovered that having separate "Chakra" and "DOM" heaps created a blind spot. The Chakra recycler didn't always have visibility into allocations on the DOM heap, and vice versa. If a pointer to a Chakra object was stored in a buffer on the DOM heap, the Chakra GC might not "see" it, leading to a UAF despite the protection.

Furthermore, killing UAF simply pushed attackers toward **Type Confusion**. MemGC ensured the memory was _there_, but it couldn't ensure the memory was being interpreted as the correct _type_.

## Lessons for Today’s Developers

If I were building a new engine today, what would I take from the MemGC era?

- **Defense in Depth**: MemGC works best when paired with modern mitigations like **[Control Flow Guard (CFG)](https://learn.microsoft.com/en-us/windows/win32/secbp/control-flow-guard)**.
- **The Precision Problem**: Conservative scanning prevents UAF but can cause memory leaks. Precise GC is the gold standard for modern systems.
- **Language Safety**: MemGC was a heroic effort to make C++ safe. Today, we should look toward **Rust** for compile-time safety guarantees.

## Final Thoughts

MemGC was a milestone in browser security. It forced attackers to abandon their favorite exploits and move into much more difficult territory. For those of us who built it, it remains a proud example of weaponizing computer science for defense.

### References:

- [MSRC: Triaging Exploitability in the MemGC Era](https://www.microsoft.com/en-us/msrc/blog/2016/01/triaging-the-exploitability-of-ieedge-crashes)
- [ZDI: Seeing Double - Exploiting a Blind Spot in MemGC](https://www.zerodayinitiative.com/blog/2018/12/17/seeing-double-exploiting-a-blind-spot-in-memgc)

> First published 4/24/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/memgc-retrospective-on-bygone-frontline.html)
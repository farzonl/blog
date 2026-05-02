---
title: "The Eigenvectors of Erid: How the Fourier Transform translated Eridian"
seoTitle: "Fourier Transform as a Universal Translator"
seoDescription: "Explore how Andy Weir envisioned Fourier transforms could turn complex alien sounds into english."
datePublished: 2026-05-01T23:30:36.078Z
cuid: cmonjq1m300fh1qjc7p495oxr
slug: the-eigenvectors-of-erid-how-the-fourier-transform-translated-eridian
canonical: https://blog.farzon.org/2026/04/the-eigenvectors-of-erid-how-fourier.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/4ab5a16c-32fd-4b7f-b201-2dd44f6fc44b.webp
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/d6ea7252-cd38-41e3-b349-fa45d6e52f92.webp
tags: matrix, eigenvectors, fast-fourier-transform, andy-weir

---

I recently saw *Project Hail Mary* and it was amaze amaze amaze! So now I'm reading the book. In the movie Grace translates Rocky's language in an unexplained way. We are left to assume some type of ML based pattern matching. In the book it is very well explained that Grace uses Fourier transform to determine which notes/chords are being "spoken" and catalogues them. I'm so happy Andy Weir didn't do any hand wavy magic ML solves this nonsense. So how the heck does this work?

Well before we answer that lets think of more practical uses. Ever wondered how your phone knows exactly which note you’re singing, or how your Wi-Fi squeezes a Netflix show through the air? It all starts with a mathematical "cheat code" involving circles and matrices.

### Meet the "Circulant Matrix" (The Infinite Loop)

Imagine a line of 8 people standing in a circle. You give a secret message to the first person. They pass it to the right, and that person passes it to the right, and so on. Eventually, the message comes right back to the start.

In math, we represent this "passing along" with a [**Circulant Matrix**](https://en.wikipedia.org/wiki/Circulant_matrix). Each row is just the previous row shifted by one spot.

```matlab

  # A simple 8x8 circulant matrix H
  H = [[4, 3, 0, 0, 0, 0, 0, 3],
       [3, 4, 3, 0, 0, 0, 0, 0],
       [0, 3, 4, 3, 0, 0, 0, 0],
       ... ]
```

Because it’s a circle, the end connects to the beginning. This matrix represents a system where every "neighbor" interacts with each other in exactly the same way, forever.

Note: A matrix is **circulant** if *each row is a cyclic right-shift of the previous row*.

So every row is obtained by rotating the previous row one step to the right with wrap-around.

### The Problem: Chords vs. Words

Unlike humans, who use vowels and consonants, Rocky speaks in chords. Imagine Rocky saying a word that sounds like a messy "hum." To a human ear (or a basic microphone), that "hum" looks like a jagged, complex wave that makes no sense.

### The Solution: Eigen-decomposition (The "Prism" Effect)

In the notebook, the core idea is that a **circulant matrix** (a system that repeats in a loop) can be broken down into its **eigenvectors** (the fundamental waves).

Grace does exactly this. He takes the complex "hum" of Rocky’s voice and runs a **Fourier Transform**. Just as our python notebook breaks a matrix into its core frequencies, Grace’s software breaks Rocky’s chords into individual notes.

*   **The Notebook Logic**: The matrix H is a messy pile of numbers, but its "DNA" consists of pure sine waves.
    
*   **The *Hail Mary* Logic**: Rocky’s "word" is a messy sound, but its "DNA" consists of distinct musical notes (frequencies).
    

### The Search for "Steady States"

In linear algebra, we look for **Eigenvectors**. Think of these as the "natural vibrations" of a system. If you pick a guitar string, it vibrates in a specific shape. That shape is an eigenvector.

When we runs `np.linalg.eig(H)` in the notebook, We are asking: *"Hey Matrix, if I send a signal through this circle, what patterns will stay the same shape as they move around?"*

### The "Fourier Basis" as a Universal Translator

Because physics is the same everywhere in the universe, the way sound waves vibrate follows the same mathematical rules on Earth as they do on Rocky’s home planet, Erid. The Fourier Transform is a "universal constant." Grace doesn't need to know *what* Rocky is saying to recognize the *math* of the sound. By identifying the specific frequencies (the eigenvalues), Grace can assign a human word to a specific combination of notes.

### The Big Reveal: Waves are the Answer

Here is where the magic happens. It turns out that for *any* circulant matrix no matter what numbers you put in you will find that the "natural vibrations" (eigenvectors) are always **Sine** and **Cosine** waves.

which means waves are the **Fourier Basis**. Said another way the eigenvectors of these matrices are the Fourier basis which are a set of standard waves that never change.

When we say we are "doing a Fourier Transform," we are actually breaking a messy signal down into these clean, circular vibrations. The math shows us that waves are the only patterns that can travel around a circle without changing their fundamental shape.

### Turning Math into Conversation

This ends up being such a beautiful homage to Carl Sagan's Contact because its yet another example of why math is the first if not universal language. Once Grace uses the Fourier Transform to isolate the notes, he builds a lookup table:

*   **Input**: Alien audio signal.
    
*   **Process**: Apply Fourier Transform (extract the "eigen-notes").
    
*   **Output**: Distinct frequencies that can be mapped to English.
    

Without the ability to perform this decomposition the same one demonstrated in the `np.linalg.eig(H)` function of the notebook—Grace would just hear noise. By treating the sound as a mathematical object that can be decomposed, he turns a literal alien "vibration" into a collaboration that saves two planets.

### Why should you care?

So far I've told you why it was cool for the book\\movie, but why should you care? By finding the **Eigenvalues** (the strength of each vibration), we can see which frequencies are the "loudest" in our data.

*   **In Music**: The eigenvalues tell us which notes are being played.
    
*   **In Images**: They tell us where the sharp edges are versus the blurry backgrounds.
    
*   **In Nature**: They explain how heat spreads or how waves move through the ocean.
    

### The Takeaway

The Fourier Transform isn't just a scary formula in a textbook. It is the mathematical proof that the universe loves to move in circles. By looking at a simple matrix, we can see that complex signals are just a choir of simple waves singing together.

**Want to try it yourself?** [Grab my Python notebook](https://gist.github.com/farzonl/d2508de79bf1deb7680b085fcca8ee00), build a circulant matrix, and watch the waves emerge from the numbers!

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg align="center")](https://colab.research.google.com/gist/farzonl/d2508de79bf1deb7680b085fcca8ee00/fourier.ipynb)

> Originally published April 8th on [blog.farzon.org](https://blog.farzon.org/2026/04/the-eigenvectors-of-erid-how-fourier.html)
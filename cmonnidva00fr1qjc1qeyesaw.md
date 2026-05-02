---
title: "To the Moon and Back: How a fundamental Algorithm is Navigating the Artemis Missions"
seoTitle: "Kalman Filters Powering NASA’s Artemis Navigation"
seoDescription: "Explore how NASA used Extended Kalman Filters to navigate the Artemis missions and learn to build one in Python from scratch."
datePublished: 2026-05-02T01:16:37.177Z
cuid: cmonnidva00fr1qjc1qeyesaw
slug: to-the-moon-and-back-how-a-fundamental-algorithm-is-navigating-the-artemis-missions
canonical: https://blog.farzon.org/2026/04/to-moon-and-back-how-fundamental.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/9dc62df1-4794-4947-9a87-272b2ab883f9.jpg
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/cc44afbf-1f44-4805-bcaf-915a61b28dc8.jpg
tags: nasa, google-colab, kalman-filter, python-notebook, artemis

---

> first published April 10th on [blog.farzon.org](https://blog.farzon.org/2026/04/to-moon-and-back-how-fundamental.html)

I have been so energized with all the cool space news in 2026. It has been one of the few things keeping me sane 🤣. So today I'm going to walk through a paper I just read as I try to consume as much space news as possible.

Have you ever wondered how a spacecraft, hurtling through the vacuum of space at thousands of miles per hour, knows exactly where it is? [Well NASA answered that question in 2023](https://ntrs.nasa.gov/citations/20230000548) in the paper _Extended Kalman Filter Performance on the Artemis-1 Mission_.

![Orion Shuttle in space](https://blogger.googleusercontent.com/img/a/AVvXsEhK-yKLBoFQFzoa-ZJXFTV-a3I5irGSbmfn3Sr8z9YfptcFayUOu4mkuo1yqqNchJPo2yTi_e89nW25Zr6LECKLlvdPAZbtLXTpz74NfX6ya8WdaH4HqS73FHlidO0gc9x009wll_iQ62weRrzXWZdp1_JWGDujbS__iTBzYkL7zCdWSubokRqjLHqEjug)
  

 The secret is the [Kalman Filter](https://en.wikipedia.org/wiki/Kalman_filter), specifially the **Extended Kalman Filter (EKF)**. During the historic Artemis missions, this algorithm was the silent pilot, running from the moment the engines ignited until the capsule splashed down in the Pacific. In this post, we’re going to peel back the curtain on how NASA used this algorithm to navigate and how you can build one yourself.

## What is a Kalman Filter and How does it work?

Imagine trying to track a car in a tunnel. You have a math model (the car’s speed) and noisy data (a fuzzy GPS signal). A Kalman Filter is like a super-smart judge that looks at both and decides: _"The GPS is a bit jumpy right now, so I'll trust the math model more,"_ or _"The car just hit a speed bump, I should trust the GPS."_

In our Python Notebook, we simulate this exact sceanrio with a vehicle moving in 1D. While the Artemis mission used 3D vectors and complex gravity models, the core logic is identical to this snippet. Let’s break down the two-step dance of the Kalman Filter.

![1D Kalman filter](https://blogger.googleusercontent.com/img/a/AVvXsEhhwLQ875obtECJr7oPS7yU1_pIO_N4naR0xwueI8wrlQdBh4z2IfdZF87SS03dY_vXQsncSHmHiDbf2cQP16qemDJTz5oI9mkUOiYdX3wZlKqmWNTZM0woz3uCjiIY0y579ltYCtmZefYHJVOzg28F_iKOclelm68Ky3CcorhiVu4KwwQ1itkSXbhnF_o)

### Phase 1: State Propagation (The Guess)

Before looking at any sensors, the filter uses physics to predict where the spacecraft _should_ be. In the NASA paper, this is called "propagating the state."

```python
# x_pred: New Position estimate
# p_pred: New Uncertainty (Covariance)
x_pred = x + v * dtp_pred = p + q
```

> _NASA Fun Fact:_ Artemis-1 used IMU data to drive this prediction at 40Hz, ensuring the spacecraft never "stopped" navigating.

### Phase 2: The Measurement Update (The Correction)

Now we compare our guess to the sensor (GPS or Optical Nav). The difference between them is the  **Residual**. The **Kalman Gain (k)** decides how much we trust that error.

```python
# Calculate how much to trust the sensor vs. the math
k_gain = p_pred / (p_pred + r)
# Correct the position using the measurement residual
x = x_pred + k_gain * (measurement - x_pred)
# Update our confidence for the next loop
p = (1 - k_gain) * p_pred
```

> _NASA Fun Fact:_ The Artemis paper highlights "Measurement Underweighting." If the measurement is too far from the prediction, the filter "dampens" the gain so the spacecraft doesn't make jerky, dangerous movements.

## NASA's Ultimate "Guess and Check" System

The last time humanity went to the moon was the Apollo missions back in the 1960s. The computers were less powerful than a Casio calculator. Navigation often required ground crews to "beam" coordinates up to the astronauts. This is the first time we have a space craft with he power to do navigation on its own and Orion does not disappoint.

The Artemis-1 Orion spacecraft used this exact "Predict-Update" cycle, but on a massive scale. As detailed in the mission architecture, Orion ran four specialized filters:

1. **Atmospheric Filter (ATMEKF)**: Handled the shaky, high-stakes launch and re-entry.
2. **Earth Orbit Filter (EOEKF)**: Used GPS satellites to stay on track near home.
3. **Cislunar Filter (CLEKF)**: Took over when GPS faded, using Optical Navigation (taking photos of the Earth and Moon) to find its way.
4. **Attitude Filter (ATTEKF)**: Used the stars to ensure the craft was always pointing in the right direction.

Breaking this down into our Predict update  cycle 

- **The Prediction**: Instead of the simple linear motion in our Python script, Orion used high-fidelity gravity models and IMU data.
- **The Update**: When near Earth, it used GPS. When near the Moon, it used _Optical Navigation (OpNav)_ which is literally taking photos of the Moon’s horizon to calculate its position.

### **What makes an Extended Kalman Filter (EKF)?**

If you look at the **Python notebook**, we used a "Standard" Kalman Filter. But the NASA paper explains that for a Moon mission, standard isn't enough. They used two critical upgrades:

1. **UDU Factorization**: In our code, we calculate the "Uncertainty Matrix" (P). In space, rounding errors in a computer can make that math "break." NASA uses UDU Factorization to break the matrix into pieces that are mathematically impossible to break, ensuring the navigation never "diverges."

2. **The Multiplicative EKF**: While our notebook handles 1D motion, Orion has to track 3D rotation. NASA uses a "Multiplicative" approach for attitude, meaning they track the _error_ in rotation and "rectify" it after every measurement.

## From Your Laptop to the Lunar Surface

The code in our Python notebook is the "Hello World" of the same system that just returned humanity to the lunar vicinity. By understanding how to balance a "prediction" with a "measurement," you are using the same logic that NASA engineers used to certify Orion for crewed flight.

![Artemis blasting off](https://blogger.googleusercontent.com/img/a/AVvXsEjul_egujtCUHdcEVI5PaRbFmb5CtpUCJ05rndgN4OIrGddKKMfHHWCDKW39mf7JdaAitxh_kV0y3YtgrmXciY_lgoWQSeO3i6oOGY6AEgG5OqzL7fcGRuVaUwZzsROtEdq4mm-dXk1HZTXp_qa-qJqavhPM7rpbszH4e7Hzx7Ya-qgM7iHE5wx7Z9rQuA)

Whether it’s a simple simulation on your screen or a multi-billion dollar mission to the Moon, the Kalman Filter is the tool that turns the "noise" of the universe into a clear path forward. 

Try out the tutorial here: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/gist/farzonl/ed122cd3686f758b7fc1089c18576893/kalman_filter.ipynb)

* * *

**References:**

- _Sullivan, J. & D’Souza, C. (2023). "Extended Kalman Filter Performance on the Artemis-1 Mission." AAS 23-053._

- _Implementation: [See the git gist Python Notebook for the 1D Kalman Filter Simulation](https://gist.github.com/farzonl/ed122cd3686f758b7fc1089c18576893)._
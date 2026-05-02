---
title: "The Geometry of Dragon Ball: A Deep Dive into SDFs and Raymarching"
seoTitle: "Dragon Ball Shader Breakdown: GLSL & Raymarching"
seoDescription: "Explore the GLSL math behind a 4-Star Dragon Ball shader. This breakdown covers 2D SDFs, raymarching, caustics, and complex 3D optical physics."
datePublished: 2026-05-02T18:55:46.333Z
cuid: cmoopcgpn00b71qgwaksy1z2p
slug: dragonball-shader
canonical: https://blog.farzon.org/2026/05/the-geometry-of-dragon-ball-deep-dive.html
cover: https://cdn.hashnode.com/uploads/covers/69f30a5b909e64ad078554d7/e7f816ae-c4ad-455d-a15a-d3ce628428a4.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/210aa450-c7c4-4e5e-8f62-cb38be336db4.png
tags: 3d, 2d, sdf, glsl, reflection, shader, shadertoy, caustics, light-refraction, raymarching

---

Today we are deconstructing the 4-Star Dragon Ball shader (available on [Shadertoy here](https://www.shadertoy.com/view/7c2SWc)). This shader is a good example of using 2D math to drive 3D optical effects. To understand it, we must reference the fundamental building blocks provided by [**Inigo Quilez**](https://iquilezles.org/).

This shader is more than just a piece of fan art. I built this [4-Star Dragon Ball shader](https://www.shadertoy.com/view/7c2SWc) as a bridge between simple 2D shapes and complex 3D optics. If you’re looking for an introductory shader to study, this is a perfect playground because it moves past "drawing a box" and dives into how light behaves. What makes this unique as a case study is the Double March. We are treating the sphere as a physical lens. Once you hit the "glass," the shader calculates a bent path and starts a second march inside. It’s my attempt at playing with coordinate transformation.

![The 4 star ball with Caustics](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgf-d4EdI0mI9bmQ-bSkQzmLyn35eanCLYKBOS4Eg4fbCrWg0sQxhWafOlqlzKviKLhjy0_oY0k4venvcYyBviO9fNyoLj0zCc-eUbH55Ub-sGyHL8FUzDZLtyjxmpn7_xrjhCn8XyJtXsonrPAA4eTW6HnGok3VbFB8W8YX_QNeQHScJAO4b-lYwYuW2c/s320/image.png align="center")

## 1\. Defining the Stars (2D SDFs)

The internal stars are defined using 2D Signed Distance Functions. an SDF is a function that takes a point in space and returns the distance to the nearest surface.

*   If the distance is **positive** , you are outside the object.
    
*   If the distance is **negative** , you are inside.
    
*   If it is **zero** , you are exactly on the surface.
    

This allows us to define perfect geometric shapes without textures. The specific star logic used here, `sdStar5`, is a direct implementation of Quilez's formulas found in his [2D Distance Functions](https://iquilezles.org/articles/distfunctions2d/) article.

```glsl
// Helper to map the 4 stars in 2D
float map4Stars2D(vec2 p) {
    float offset = 0.22;
    float radius = 0.15;
    float d1 = sdStar5(p - vec2(offset, offset), radius, 0.45);
    float d2 = sdStar5(p - vec2(-offset, offset), radius, 0.45);
    float d3 = sdStar5(p - vec2(offset, -offset), radius, 0.45);
    float d4 = sdStar5(p - vec2(-offset, -offset), radius, 0.45);
    return min(min(d1, d2), min(d3, d4)); // Union of all four stars
}
```

## 2\. Extrusion and 3D Modeling

To turn these flat stars into 3D geometry inside the sphere, we use [**Extrusion**](https://ansyshelp.ansys.com/public/account/secured?returnurl=/Views/Secured/corp/v251/en/poly_pftut/pf_train_3d-extru.html). By combining the 2D distance with the Z-axis, we create a volumetric shape. This technique, along with the rounding (beveling) seen in the code, is detailed in Quilez’s [3D Distance Functions](https://iquilezles.org/articles/distfunctions/) article.

```glsl
float mapStars3D(vec3 p) {
    float d2d = map4Stars2D(p.xy);
    vec2 w = vec2(d2d, abs(p.z) - 0.04); // abs(p.z) defines thickness
    return min(max(w.x, w.y), 0.0) + length(max(w, 0.0)) - 0.015;
}
```

## 3\. The Raymarching Engine

How does the computer "see" these math functions? For a short intro think of it this way. Since we don't have triangles, we can't use standard rasterization. Instead, we use [**Raymarching**](https://en.wikipedia.org/wiki/Ray_marching).

Imagine firing a ray from your eye through every pixel. Instead of moving in tiny constant steps, the ray "leaps" forward by the distance returned by our SDF. Because the SDF tells us the distance to the *nearest* object, we know it's safe to move that far without hitting anything. We repeat this until the distance is nearly zero (a hit!) or we've gone too far (a miss).

For a full deep dive into the algorithm that powers almost every modern Shadertoy, read Quilez's [Raymarching Distance Fields](https://iquilezles.org/articles/raymarchingdf/).

In our code, this is the loop that "steps" through space until it finds the sphere or the floor.

```glsl
for(int i = 0; i < MAX_STEPS; i++) {
    p = ro + rd * dO; // Current position along the ray
    dS = map(p); // Distance to nearest object
    dO += dS.x; // "Safe" jump forward
    if(dO > MAX_DIST || abs(dS.x) < SURF_DIST) break;
}
```

## 4\. Optical Physics: Refraction & Reflection

To simulate glass, we don't just "draw" it; we simulate how light behaves at the boundary of two materials.

*   [**Fresnel Effect**](https://en.wikipedia.org/wiki/Fresnel_equations): We use the dot product of our view ray and the surface normal to decide if we see the interior (refraction) or the exterior (reflection).
    
*   **The Nested March**: When the ray enters the sphere, it bends. The code then starts a *new* internal raymarch loop. This is what allows the stars to look magnified and distorted by the glass.
    

```glsl
vec3 refractDir = refract(rd, n, 1.0 / 1.52); // Bending light into glass
// ... inside the sphere ...
float volumetricGlow = exp(-minStarDist * 18.0); // Creates that resin-like haze
refractCol += emissiveStar * volumetricGlow * 0.45;
```

## 5\. Analytical Caustics & Chromatic Aberration

The glowing star patterns on the floor are [**caustics**](https://en.wikipedia.org/wiki/Caustic_\(optics\)). **Caustics** are those dancing, wavy patterns of light you see at the bottom of a swimming pool or the bright "ring" at the base of a wine glass. They happen whenever a curved, transparent surface like water or a glass ball acts as a lens, grabbing scattered light rays and bunching them together into concentrated, glowing shapes.

![Caustics example from Wikipedia](https://upload.wikimedia.org/wikipedia/commons/8/8e/Kaustik.jpg align="center")

I achieve this by using `iSphere` to project the floor's coordinates back into the star field.

To add realism, I implement \*\*[Chromatic Aberration](https://en.wikipedia.org/wiki/Chromatic_aberration) \*\*splitting the light into R, G, and B components by shifting the UV samples slightly.

![Chromatic Aberration example from Wikipedia](https://upload.wikimedia.org/wikipedia/commons/thumb/a/aa/Chromatic_aberration_lens_diagram.svg/3840px-Chromatic_aberration_lens_diagram.svg.png align="center")

**Chromatic Aberration** is a lens effect that creates a "rainbow fringe" around sharp edges. It happens because glass doesn't bend all colors of light equally; red light bends a little less than blue light. In a shader, you mimic this by "splitting" the image into its red, green, and blue components and shifting them slightly so they don't perfectly line up, giving the scene a realistic, cinematic look.

```glsl
float shift = 0.015; 
float dR = map4Stars2D(distortedUV * (1.0 - shift));
float dG = map4Stars2D(distortedUV);
float dB = map4Stars2D(distortedUV * (1.0 + shift));
vec3 starShadow = vec3(smoothstep(0.03, -0.03, dR), ...);
```

## The "Secret Sauce": Why This Works

The uniqueness of this shader lies in its \*\*layered approach\*\*. It isn't just one distance field; it’s a coordinate-space transformation. By distorting the UVs on the floor (`distortedUV`), I simulate the way a magnifying glass bends light, creating a dynamic caustic pool that responds to the pulsing of the internal emissive stars. I was really trying my best to simulate the below image.

![Toy glass dragon ball you can buy from Amazon](https://blogger.googleusercontent.com/img/a/AVvXsEgq4QNxZLFMfz0_F4i7XaJqMOMdabDFmX7oO-QaIVJAJj52ID8ClM8hstPPAkEwiM9Qon8SkuyYX0lVVmG9gkxrF1tCB5T_EAIpH04IdB-kClIxT9jglYIILuGZcREmBy7JbV72EfmAdWs6BigPA4sifIlgTrZWNVWi8ktaSlsznEDpQ3yuHhwGvwS58k0 align="center")

## Conclusion

Writing shaders is about translating the laws of physics into the language of mathematics. By combining Inigo Quilez’s SDFs with Raymarching, you can build complex, beautiful worlds with nothing but a few lines of GLSL.

**References & Further Reading:** 

- [Raymarching Distance Fields (Quilez)](https://iquilezles.org/articles/raymarchingdf/)  
- [3D Distance Functions (Quilez)](https://iquilezles.org/articles/distfunctions/)  
- [2D Distance Functions (Quilez)](https://iquilezles.org/articles/distfunctions2d/)
- [4-Star Dragon Ball by Me (Noztol)](https://www.shadertoy.com/view/7c2SWc)
- [noztol\_shades instagram](https://www.instagram.com/noztol_shades)

> First published 5/2/2026 on [blog.farzon.org](https://blog.farzon.org/2026/05/the-geometry-of-dragon-ball-deep-dive.html)
---
title: "From Sidelined to Streamlined: Cracking the Code (Part 2)"
seoTitle: "Wrestling with XML: How to Export & Parse Apple Health Data"
seoDescription: "Learn how to stream Apple Health XML data, use LOINC codes for Heart Rate/SpO2, and parse GPX workout routes with Python."
datePublished: 2026-05-02T08:09:59.398Z
cuid: cmoo29zdx00nh1qjch2oh5rnh
slug: from-sidelined-to-streamlined-cracking-the-code-part-2
canonical: https://blog.farzon.org/2026/04/from-sidelined-to-streamlined-cracking.html
ogImage: https://cdn.hashnode.com/uploads/og-images/69f30a5b909e64ad078554d7/bcdbf661-7ca3-48d0-a362-bcdc8de63b47.png
tags: apple, python, iphone, applehealth

---

In [Part 1](https://blog.farzon.org/2026/04/from-sidelined-to-streamlined-decoding.html), I talked about the high-level "magic" happening on your wrist. How Apple uses neural networks and math to guess your VO2Max even when you’re just walking around the block. But if you're like me, "trusting papers" isn't enough. I wanted to get my hands on the raw numbers to see the "Prediction Delta" for myself

The problem? When you hit "_Export Health Data_" on your iPhone, you don’t get a clean spreadsheet. You get a giant, messy folder full of scary XML files.

![How to export Apple health data](https://blogger.googleusercontent.com/img/a/AVvXsEjVhpl3u3sXHKghuJFwixZP9Nbs7NwM7DQjImNX1-H-Lwuj9NlEEA8_9-ji8sHZHJv8enaoTaewi7PZP88VuTlZInNnsHoWIvKJZ5zOEMvLK_eI0R_eIu8bgu-gBIcbZTdMl0JoYx_LOp8SjImvhInURW72IFA3IujcPi9RE4vSrM5tLH6xGTN0zAwBxiw)
  
Here’s how I tackled the two biggest data hurdles in the export: the massive health records and the hidden workout GPS paths.

## Streaming the "Heavyweight" (The CDA File)

Inside your export, there's a file called `export_cda.xml`. This is where the "clinical" stuff lives. For example The  second-by-second Heart Rate and Blood Oxygen levels we will use to compute a VO2Max.

The catch? This file can easily be 300MB or more. If you try to open that in a normal text editor or a basic script, your computer will probably hang. To get around this, I wrote a "streaming" parser. Instead of loading the whole file at once, it reads it one tiny piece at a time, grabs what it needs, and then throws the rest away.

**The "Secret Sauce" in the Code**:

- **LOINC Codes**: I told the script to ignore everything except two specific ID numbers: `8867-4` for Heart Rate and `2710-2` for Blood Oxygen.
- **Cleaning the Mess**: Apple records Oxygen as a decimal (like `0.98`), so I had the script multiply that by 100 to make it a readable percentage.
- **Memory Management**: By using `elem.clear()`, the script stays fast and light, no matter how many years of data you've collected.

## Hunting for Your Routes (The GPX Files)

If you’ve ever looked at your workout map in the Fitness app and wondered where that data lives, it's in the `workout-routes/` folder. Every time you track a walk or run, Apple generates a `.gpx` file.

These files are great because they contain the "Environmental Kinematics"—the pace, distance, and topography that Apple uses to validate your fitness.

### **What my GPX Parser does**:  

It loops through every file in that folder and extracts more than just your GPS coordinates. It calculates:
- **The "Verticality"**: It finds the highest and lowest points to see how much of a hill you actually climbed.
- **The Pace**: It pulls the speed data directly recorded by the watch sensors.
- **The Duration**: It calculates exactly how long you were moving by comparing the first and last timestamps.

## Why do we need this data?

When these two scripts finish running, that mountain of XML files is transformed into a few clean CSV files. Now, instead of staring at raw code, I have a clear spreadsheet of my rehab walks.

I can see exactly how my heart rate reacted to that one steep hill, and more importantly, I have the "Data Diet" ready to feed into my own model.

Grab the code [here](https://gist.github.com/farzonl/8ed72ce833d91de1c44dfd8d5072bd4d).

**Coming up in Part 3:** We finally get to the fun part. I’m taking these CSVs into a Jupyter Notebook to build my own Multi-Layer Perceptron (MLP). We’re going to see if my DIY "Watch Brain" comes up with the same VO2Max scores as Apple's. Stay tuned!

> First published 4/12/26 on [blog.farzon.org](https://blog.farzon.org/2026/04/from-sidelined-to-streamlined-cracking.html)
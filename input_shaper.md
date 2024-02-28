# Input Shaper
This page is a brief overview of input shaper. If you want an in depth guide with lots of details, visit the Klipper page on Resonance Compensation and Measuring Resonances.

**Resonance Compensation**

https://www.klipper3d.org/Resonance_Compensation.html

**Measuring Resonances**

https://www.klipper3d.org/Measuring_Resonances.html

# What is Input Shaper?

Input shaper is a method of changing the motion of something such that we cancel out unwanted motion (from vibrations or resonance). In other words, we are intentionally changing the signals sent to our stepper motors to cancel out motion.
This is an open loop method meaning it isn't done in reaction to a measurement but rather done knowing that there will be a reaction.
For example, imagine you're in the ocean and you see a wave coming. You would jump because you know a wave is coming and you want to keep your head above water. That would be an open loop control. Now imagine you're doing the same with your eyes closed.
Now, you would jump in reaction to the wave when you feel it. That would be a closed loop control.

Input shaper is basically measuring the resonances of your machines and implementing a control that cancels out those resonances. Check out these two links below for very, very intuitive examples.

https://www.youtube.com/watch?v=5fOhi-LL9dU

https://www.youtube.com/watch?v=gzBhTrHv0-c

# How do we get Input Shaper?

To get input shaper we need to know what we're trying to cancel out, our resonances. Resonances can be seen in your print as repetitive patterns. These artifacts are colloquially referred to as ghosting. There are two ways of going about this.

## Manual Measurement

The first way is manual and arguably worse in all ways. In this method you need to print out some test objects with varying speeds and accelerations. You then measure the amount of ripples seen in the print and how far they extend.
This page will not cover this as I've not personally done it and because it is covered ad naseum on the Klipper page. (Resonance Compensation)

## Automatic Measurement

The next method of measurement is much simpler and more accurate but requires some setup. There are a few things we will need...

1. MCU
2. Accelerometer
3. Mounting Hardware

The MCU is a microcontroller. You can use a Raspberry Pi, a Raspberry Pi Pico, or any other microcontroller that can communicate via SPI. Other communication methods are possible but not with the accelerometer I used. See the Klipper documentation for explanations of why.

The accelerometer is a small chip (on a board) that will be measuring the resonances of our printer for us. We need to connect this to our MCU so that the MCU can record he output.

The mounting hardware is what we need to attach the accelerometer to our print head and (in the case of a bed slinger) our print bed. We need to make sure this properly alings the axes of the accelerometer to the axes of the printer.

# Setting up Input Shaper Hardware

## Raspberry Pi

## Raspberry Pi Pico

## EBB 42

# Software

# Measuring Resonances

# Configuration

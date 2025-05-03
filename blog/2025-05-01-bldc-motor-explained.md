---
slug: bldc
title: How does a BLDC motor work?
authors: carlos
tags: [Motors]
draft: false
---

# How does a BLDC motor work?

<div style={{textAlign: 'center'}}>

![BLDC Motor](/img/BLDC-Motor.gif)

</div>

It's just magnets and wires.

<!-- truncate -->

## A compass and a wire

When you run a current through a wire, you get a magnetic field around the wire.

<div style={{textAlign: 'center'}}>
![Magnetic field around a current-carrying wire](/img/faradays-law.jpg)
</div>

If I put a permanent magnet like a compass next to the wire, the magnetic field of the compass interacts with the magnetic field around the wire which causes the compass to be deflected. 

<div style={{textAlign: 'center'}}>
![Wire deflecting compass](/img/wire-deflecting-compass.gif)
</div>

This is a result of the north and south poles of each field being attracted to each other and trying to align.

A motor works in the same way.

<div style={{textAlign: 'center'}}>
![Brushless Motor](/img/BLDC-Motor.gif)
</div>

Inside of the motor there are both permanent magnets and wires. When you run a current through the wires inside the motor, the magnetic field of the wire interacts with the magnetic field of the permanent magnets -- just like the compass and wire we talked about before.

As you can see in the image above, there are two main parts of a motor: a rotor and a stator. 

The rotor is free to rotate and the stator is stationary.

The permanent magnets are attached to the rotor and the wire is wrapped around specific parts of the stator.

Now if I start to run current through the wire, I get a magnetic field that starts to interact with the magnetic field of the permanent magnets. Since the permanent magnets on the rotor are free to rotate, the rotor will rotate to try to align its magnetic field with the one created by the wire. And so, we get a rotation. Success!

This type of motor is called a brushless DC motor. 

One thing I want to point out is that in the motor image above current is always flowing through two of the wires. This is because one of the wires always interacts with the south pole (blue in the image) and the other wire interacts with the north pole (red). You can think of this as one magnetic field "pushing" while the other is "pulling" since two poles of the same polarity repel each other while poles of opposite polarity attract.

Lastly, depending on the motor design, there may also be sensors inside the motor that provide position data like hall-effect sensors or encoders.

## That's it?

This post is purposefully short because I extracted it from a different post that talks about a method for controlling motors called [Field Oriented Control (FOC)](2025-05-03-foc.md).

However, the explanation was broad enough that it applied to all BLDC motors so I decided to make it its own post.

Hopefully this makes the inner workings of BLDC motors a bit less mysterious.

![PMSM Exploded View](/img/pmsm-exploded-view.jpg)
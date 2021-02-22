---
layout: post
title: Ballistics Models 
tags: [Visualization, Physics Engine]
excerpt_separator: <!--more-->
---

<!--more-->


# Ballistics Models

> This is the summary of Chapter 4 of Ian Millington's Game Physics Engine Development: How to Build A Robust Commercial-Grade Physics Engine For Your Game. This site is for EDUCATIONAL PURPOSES ONLY.

## Overview

One of the most common applications of physics simulation in games is to model ballistics. This has been the case for two decades, predating the current vogue for physics engines.

Each weapon has a characteristic muzzle velocity, the speed at which the projectile is emitted from the weapon. This will be very fast for a laser bolt, and probably considerably slower for a fireball. For each weapon, the muzzle velocity used in the game is unlikely to be the same as its real-world equivalent.

## Setting Projectile Properties

The muzzle velocity for the slowest real-world guns is on the order of 250 m/s​, whereas tank rounds designed to penetrate armor plate can move at 1800 m/s​.  The muzzle velocity of an energy weapon such as a laser would be the speed of light: 300,000,000 m/s​. Instead, if we want the projectile's motion to be visible, we use muzzle velocities that are in the region of 5 to 25 m/s, for a human-scale game. The equation that links energy, mass, and speed is 
$$
e = ms^2
$$
If we want to keep the same energy, we can work out the change in mass for a known change in speed as follow:
$$
\Delta m  =(\Delta s)^2
$$
For gravity, if we are using a higher gravity coefficient in the game, it will make the ballistic trajectory far too severe: well-aimed projectiles will hit the ground only a few meters in front of the character. To avoid this, we lower the gravity. For a known change in speed, we can work out a “realistic” gravity value using the formula,
$$
g_{\text{bullet}}=\frac{1}{\Delta s}g_{\text{normal}}
$$
where $g_{\text{normal}}$ is the gravity you'd expect of the particle was traveling at full speed.

## Implementation

Let's design a program that gives you the choice of four weapons: a pistol, an artillery piece, a fireball, and a laser gun. When you click the mouse, a new round is fired. The code that creates a new round and fires it looks like this:

```C++
// Set the properties of the particle.
switch(currentShotType)
{
case PISTOL:
	shot->particle.setMass(2.0f); // 2.0kg
	shot->particle.setVelocity(0.0f, 0.0f, 35.0f); // 35m/s
	shot->particle.setAcceleration(0.0f, -1.0f, 0.0f);
	shot->particle.setDamping(0.99f);
	break;
case ARTILLERY:
	shot->particle.setMass(200.0f); // 200.0kg
	shot->particle.setVelocity(0.0f, 30.0f, 40.0f); // 50m/s
	shot->particle.setAcceleration(0.0f, -20.0f, 0.0f);
	shot->particle.setDamping(0.99f);
	break;
case FIREBALL:
	shot->particle.setMass(1.0f); // 1.0kg - mostly blast damage
	shot->particle.setVelocity(0.0f, 0.0f, 10.0f); // 5m/s
	shot->particle.setAcceleration(0.0f, 0.6f, 0.0f); // Floats up
	shot->particle.setDamping(0.9f);
	break;
case LASER:
	// Note that this is the kind of laser bolt seen in films,
	// not a realistic laser beam!
	shot->particle.setMass(0.1f); // 0.1kg - almost no weight
	shot->particle.setVelocity(0.0f, 0.0f, 100.0f); // 100m/s
	shot->particle.setAcceleration(0.0f, 0.0f, 0.0f); // No gravity
	shot->particle.setDamping(0.99f);
	break;
}
// Set the data common to all particle types.
shot->particle.setPosition(0.0f, 1.5f, 0.0f);
shot->startTime = TimingData::get().lastFrameTimestamp;
shot->type = currentShotType;
// Clear the force accumulators.
shot->particle.clearAccumulator();
```

Note that each weapon configures the particle with a different set of values. The surrounding code is skipped here for brevity (you can refer to the source code to see how and where variables and data types are defined).

```c++
// Update the physics of each particle in turn.
for (AmmoRound *shot = ammo; shot < ammo+ammoRounds; shot++)
{
	if (shot->type != UNUSED)
	{
		// Run the physics.
		shot->particle.integrate(duration);
		// Check to see if the particle is now invalid.
		if (shot->particle.getPosition().y < 0.0f ||
			shot->startTime+5000 < TimingData::get().lastFrameTimestamp||
			shot->particle.getPosition().z > 200.0f)
		{
			// We simply set the shot type to be unused, so the
			// memory it occupies can be reused by another shot.
			shot->type = UNUSED;
        }
    }
}
```

It simply calls the integrator on each particle in turn. After it has updated the particle, it checks whether the particle is below zero height, in which case it is removed. The particle will also be removed if it is a long way from the firing point (100 m), or if it has been in flight for more than 5 s. In a real game you would use some kind of collision detection system to check if the projectile had collided with anything. Additional game logic could then be used to reduce the hit points of the target character, or add a bullet-hole graphic to a surface.

(I will upload a gif demo later.)
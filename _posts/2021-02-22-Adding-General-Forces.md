---
layout: post
title: Fireworks
tags: [Visualization, Physics Engine, Drag Force, Gravity]
excerpt_separator: <!--more-->
---
A Physics engine cope with multiple different forces acting at the same time. 

<!--more-->


# Adding General Forces

> This is the summary of Chapter 5 of Ian Millington's Game Physics Engine Development: How to Build A Robust Commercial-Grade Physics Engine For Your Game. This site is for EDUCATIONAL PURPOSES ONLY.

### D'Alembert's Principle

For particles, D’Alembert’s principle implies that if we have a set of forces acting on an object, we can replace all those forces with a single force, which is calculated by:

$$
f = \sum_{i}f_i
$$

In other words, we simply add the forces together using vector addition, and we apply the single force that results.

To make use of this result, we use a vector as a force accumulator. In each frame we zero the vector and add each applied force in turn using vector addition. The final value will be the resultant force to apply to the object. We add a method to the particle that is called at the end of each integration step to clear the accumulator of the forces that have just been applied:

```c++
class Particle
{
	// ... Other Particle code as before ...
	/**
	* Holds the accumulated force to be applied at the next
	* simulation iteration only. This value is zeroed at each
	* integration step.
	*/
	Vector3 forceAccum;
	/**
	* Clears the forces applied to the particle. This will be
	* called automatically after each integration step.
	*/
	void clearAccumulator();
};
```

```c++
void Particle::integrate(real duration)
{
	// We don’t integrate things with infinite mass.
	if (inverseMass <= 0.0f) return;
	assert(duration > 0.0);
	// Update linear position.
	position.addScaledVector(velocity, duration);
	// Work out the acceleration from the force.
    Vector3 resultingAcc = acceleration;
	resultingAcc.addScaledVector(forceAccum, inverseMass);
	// Update linear velocity from the acceleration.
	velocity.addScaledVector(resultingAcc, duration);
	// Impose drag.
      velocity *= real_pow(damping, duration);
	// Clear the forces.
	clearAccumulator();
}
void Particle::clearAccumulator()
{
    forceAccum.clear();
}
```

We then add a method that can be called to add a new force into the accumulator:

```c++
class Particle
{
	// ... Other particle code as before ...
	/**
	* Adds the given force to the particle to be applied at the
	* next iteration only.
	*/
	void addForce(const Vector3 &force);
};
```

```c++
void Particle::addForce(const Vector3 &force)
{
	forceAccum += force;
}
```

This accumulation stage needs to be completed just before the particle is integrated. All the forces that apply need to have a chance to add themselves to the accumulator. We can do this by manually adding code to our frame update loop that adds the appropriate forces. This is appropriate for forces that will only occur for a few frames.

### Implementation

The interface for the force generator only needs to provide a current force. This can then be accumulated and applied to the object. The interface we will use looks like this:

```c++
/**
* A force generator can be asked to add a force to one or more
* particles.
*/
class ParticleForceGenerator
{
public:
	/**
	* Overload this in implementations of the interface to calculate
	* and update the force applied to the given particle.
	*/
	virtual void updateForce(Particle *particle, real duration) = 0;  
}
```

```c++
/**
* Holds all the force generators and the particles that they apply to.
*/
class ParticleForceRegistry
{
protected:
	/**
	* Keeps track of one force generator and the particle it
	* applies to.
	*/
  struct ParticleForceRegistration
  {
      Particle *particle;
      ParticleForceGenerator *fg;
  };  
    /**
    * Holds the list of registrations.
    */
    typedef std::vector<ParticleForceRegistration> Registry;
	Registry registrations;
public:
	/**
	* Registers the given force generator to apply to the
	* given particle.
	*/
    void add(Particle* particle, ParticleForceGenerator *fg);
    /**
	* Removes the given registered pair from the registry.
	* If the pair is not registered, this method will have
	* no effect.
	*/
	void remove(Particle* particle, ParticleForceGenerator *fg);
    /**
	* Clears all registrations from the registry. This will
	* not delete the particles or the force generators
	* themselves, just the records of their connection.
	*/
	void clear();
    /**
	* Calls all the force generators to update the forces of
	* their corresponding particles.
	*/
	void updateForces(real duration);
};
```

At each frame, before the update is performed, the force generators are all called. They will hopefully be adding forces to each particle’s accumulator. Later these accumulated forces are used to calculate each particle’s acceleration:

```c++
#include <cyclone/pfgen.h>
using namespace cyclone;
void ParticleForceRegistry::updateForces(real duration)
{
	Registry::iterator i = registrations.begin();
	for (; i != registrations.end(); i++)
	{
		i->fg->updateForce(i->particle, duration);
	}
}
```

### A Gravity Force Generator

We can replace our previous implementation of gravity by a force generator. Rather than special-case code to apply a constant acceleration at each frame, gravity is represented as a regular force generator attached to each particle. The implementation looks like this:

```c++
/**
* A force generator that applies a gravitational force. One instance
* can be used for multiple particles.
*/
class ParticleGravity : public ParticleForceGenerator
{
	/** Holds the acceleration due to gravity. */
	Vector3 gravity;
public:
	/** Creates the generator with the given acceleration. */
    ParticleGravity(const Vector3 &gravity);
    /** Applies the gravitational force to the given particle. */
	virtual void updateForce(Particle *particle, real duration);
};
```

```c++
void ParticleGravity::updateForce(Particle* particle, real duration)
{
	// Check that we do not have infinite mass.
	if (!particle->hasFiniteMass()) return;
	// Apply the mass-scaled force to the particle.
	particle->addForce(gravity * particle->getMass());
}
```

Note that the force is calculated based on the mass of the object passed into the `updateForce`method. The only piece of data stored by the class is the acceleration due to gravity. One instance of this class could be shared among any number of objects.

### A Drag Force Generator

We could also implement a force generator for drag. Drag is a force that acts on a body and depends on its velocity. A full model of drag involves more complex mathematics than we can easily perform in real time. Typically, in game applications we use a simplified model of drag where the drag action on a body depends on the speed of the object and the square of its speed,

$$
f_{drag} = -\hat{\dot{\bf{p}}}(k_1|\hat{\dot{\bf{p}}}| + k_2|\hat{\dot{\bf{p}}}|^2)
$$

where $$k_1$$ and $$k_2$$ are two constants that characterize how strong the drag force is - they are usually called the "drag coefficients" and they depend on both the object and the type of drag behind simulated. $$\hat{\dot{\bf{p}}}$$ is the normalized velocity of the particle. The implementation for the drag generator looks like this:


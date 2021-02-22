---
layout: post
title: Fireworks
tags: [Visualization, Physics Engine, Fireworks]
excerpt_separator: <!--more-->
---
Simple implementation of fireworks.

<!--more-->


# Fireworks

> This is the summary of Chapter 4 of Ian Millington's Game Physics Engine Development: How to Build A Robust Commercial-Grade Physics Engine For Your Game. This site is for EDUCATIONAL PURPOSES ONLY.

### The Fireworks Data

First, we need to know what kind of particle it represents. Fireworks consist of a number of payloads: the initial rocket may burst into several lightweight mini-fireworks that explode again after a short delay. We represent the type of firework by an integer value.

Second, we need to know the age of the particle. Fireworks consist of a chain reaction of pyrotechnics with carefully timed fuses. A rocket will first ignite its rocket motor, and then after a short time of flight, the motor will burn out as the explosion stage detonates. This may scatter additional units, each of which has a fuse of the same length, allowing the final bursts to occur at roughly the same time (not exactly the same time, however, as that would look odd). To support this, we keep the age for each particle and update it at each frame.

```c++
/**
	* Fireworks are particles, with additional data for rendering and
	* evolution.
	*/
class Firework : public cyclone::Particle
{
public:
    /** Fireworks have an integer type, used for firework rules. */
	unsigned type;
	/**
	* The age of a firework determines when it detonates. Age gradually
	* decreases; when it passes zero the firework delivers its payload.
	* Think of age as fuse left.
	*/
	cyclone::real age;
};
```

### Fireworks Rules

To define the effect of a composite firework, which may be made up of several of our firework effects, we need to be able to specify how one type of particle changes into another. We do this as a set of rules: for each firework type we store an age, and a set of data for additional fireworks that will be spawned when the age is passed. This is held in a rules data structure with the following form:

```c++
/**
* Firework rules control the length of a firework’s fuse and the
* particles it should evolve into.
*/
struct FireworkRule
{
	/** The type of firework that is managed by this rule. */
	unsigned type;
	/** The minimum length of the fuse. */
	cyclone::real minAge;
	/** The maximum length of the fuse. */
	cyclone::real maxAge;
	/** The minimum relative velocity of this firework. */
    cyclone::Vector3 minVelocity;
	/** The maximum relative velocity of this firework. */
	cyclone::Vector3 maxVelocity;
	/** The damping of this firework type. */
	cyclone::real damping;
	/**
	* The payload is the new firework type to create when this
	* firework’s fuse is over.
	*/
	struct Payload
	{
        /** The type of the new particle to create. */
		unsigned type;
        /** The number of particles in this payload. */
		unsigned count;
		/** Sets the payload properties in one go. */
		void set(unsigned type, unsigned count)
		{
			Payload::type = type;
			Payload::count = count;
		}
    };
    /** The number of payloads for this firework type. */
	unsigned payloadCount;
    /** The set of payloads. */
	Payload *payloads;
};
```

Rules are provided in the code, and defined in a single function that controls the behavior of all possible fireworks. The following is a sample of that function:

```c++
void FireworksDemo::initFireworkRules()
{
	// Go through the firework types and create their rules.
	rules[0].init(2);
	rules[0].setParameters(
        1, // type
		0.5f, 1.4f, // age range
		cyclone::Vector3(-5, 25, -5), // min velocity
		cyclone::Vector3(5, 28, 5), // max velocity
		0.1 // damping
		);
    rules[0].payloads[0].set(3, 5);
	rules[0].payloads[1].set(5, 5);
    
    rules[1].init(1);
	rules[1].setParameters(
        2, // type
		0.5f, 1.0f, // age range
		cyclone::Vector3(-5, 10, -5), // min velocity
		cyclone::Vector3(5, 20, 5), // max velocity
		0.8 // damping
		);
    rules[1].payloads[0].set(4, 2);
    
    rules[2].init(0);
	rules[2].setParameters(
        3, // type
		0.5f, 1.5f, // age range
		cyclone::Vector3(-5, -5, -5), // min velocity
		cyclone::Vector3(5, 5, 5), // max velocity
		0.1 // damping
		);
    // ... and so on for other firework types ...
}
```

### The Implementation

In each frame, each firework has its age updated, and is checked against the rules. If its age is past the threshold, then it will be removed and more fireworks will be created in its place (the last stage of the chain reaction spawns no further fireworks). The firework update function now looks like this:

```c++
class Firework : public cyclone::Particle
{
public:
	/**
	* Updates the firework by the given duration of time. Returns true
	* if the firework has reached the end of its life and needs to be
	* removed.
	*/
	bool update(cyclone::real duration)
	{
		// Update our physical state.
		integrate(duration);
		// We work backward from our age to zero.
		age -= duration;
		return (age < 0) || (position.y < 0);
	}
};
```

Note that if we don’t have any spare firework slots when a firework explodes into its components, then not all the new fireworks will be initialized. In otherwords, when resources are tight, older fireworks get priority. This allows us to put a hard limit on the number of fireworks being processed, which can avoid having the physics slow down when things get busy.

```c++
struct FireworkRule
{
	/**
	* Creates a new firework of this type and writes it into the given
	* instance. The optional parent firework is used to base position
	* and velocity on.
	*/
	void create(Firework *firework, const Firework *parent = NULL) const
	{
		firework->type = type;
        firework->age = random.randomReal(minAge, maxAge);
		cyclone::Vector3 vel;
		if (parent) {
			// The position and velocity are based on the parent.
			firework->setPosition(parent->getPosition());
			vel += parent->getVelocity();
		}
		else
        {
			cyclone::Vector3 start;
			int x = (int)random.randomInt(3) - 1;
			start.x = 5.0f * cyclone::real(x);
			firework->setPosition(start);
		}
        
        vel += random.randomVector(minVelocity, maxVelocity);
		firework->setVelocity(vel);
        
        // We use a mass of 1 in all cases (no point having fireworks
		// with different masses, since they are only under the influence
		// of gravity).
        firework->setMass(1);
		firework->setDamping(damping);
		firework->setAcceleration(cyclone::Vector3::GRAVITY);
		firework->clearAccumulator();
    }
};

void FireworksDemo::create(unsigned type, const Firework *parent)
{
		// Get the rule needed to create this firework.
		FireworkRule *rule = rules + (type - 1);
		// Create the firework.
		rule->create(fireworks+nextFirework, parent);
		// Increment the index for the next firework.
		nextFirework = (nextFirework + 1) % maxFireworks;
}
```


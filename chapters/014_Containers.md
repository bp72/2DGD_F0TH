{{pagebreak}}

Useful Containers and Classes
=============================

:::::: {.epigraph author="Anonymous"}
Eliminate effects between unrelated things. Design components that are self-contained, independent, and have a single, well-defined purpose.
::::::

In this chapter we will introduce some useful data containers and classes that could help you solve some issues or increase your game's maintainability and flexibility.

Resource Manager
-----------------

A useful container is the "resource manager", which can be used to store and manage textures, fonts, sounds and music easily and efficiently.

A resource manager is usually implemented via generic programming, which helps writing DRY code, and uses search-efficient containers like hash tables, since we can take care of loading and deletion during loading screens.

First of all, we need to know how we want to identify our resource; there are many possibilities:

- **An Enum:** this is usually implemented at a language-level as an "integer with a name", it's light but every time we add a new resource to our game, we will need to update the enum too;
- **The file path:** this is an approach used to make things "more transparent", but every time a resource changes place, we will need to update the code that refers to such resource too;
- **A mnemonic name:** this allows us to use a special string to get a certain resource (for instance "skeleton_spritesheet"), and every time our resource folder changes, we will just need to update our loading routines (similarly to the Enum solution).

Secondarily, we need to make sure that the container is **thread-safe** (see more about multithreading in the [multithreading section](#multithreading)), since we will probably need to implement a threaded loading screen (see how to do it [here](#loadingscreen)) to avoid our game locking up during resource loading.

{{placeholder}}

<!-- TODO: An associative container that contains pairs (ENUM, ITEM) used to store and retrieve objects for the game -->

Animator
---------

This can be a really useful component to encapsulate everything that concerns animation into a simple and reusable package.

The animation component will just be updated (like the other components) and it will automatically update the frame of animation according to an internal timer, usually by updating the coordinates of the rectangle that defines which piece of a sprite sheet is drawn.

{{placeholder}}

<!-- TODO: A class that yelds frames, with an internal counter and all the necessary facilitations for animating a sprite -->

Finite State Machine
---------------------

A finite state machine is a model of computation that represents an abstract machine with a finite number of possible states but where one (or a finite number) of states can be "in execution" at a given time.

We can use a finite state machine to represent the status of a player character, like in the following diagram:

![Diagram of a character's state machine](./images/containers/Character_SM.svg){width=70%}

Each state machine is made out of two main elements:

- **states** which define a certain state of the system (for the diagram above, the states are: Idle, Crouching, Jumping and Stomping);
- **transitions** which define a condition and the change of state of the machine (for the diagram above there are two "Press Down" transitions, one "Release Down" and one "Press Space")

State machines are really flexible and can be used to represent a menu system, for instance:

![Diagram of a menu system's state machine](./images/containers/Menu_SM.svg){width=70%}

In this more convoluted diagram we can see how pressing a certain button or clicking a certain option can trigger a state change.

Each state can be created so it has its own member variables and methods: in a menu system it can prove useful to have each state have its own `update(dt)` and `draw()` functions to be called from the main game loop, to improve on the code readability and better usage of the nutshell programming principle.

{{placeholder}}

<!-- TODO: A simple finite state machine that allows to change states, useful for menus and stuff -->

Menu Stack
-----------

Although menus can be represented via a finite state machine, the structure of an User Interface (UI) is better suited for another data model: the stack (or rather, something similar to a stack).

A stack allows us to code some other functions in an easier way, for instance we can code the "previous menu" function by just popping the current menu out of the stack; when we access a new menu, we just push it into the menu stack and the menu stack will take care of the rest.

Unlike the stacks we are used to, the menu stack can also be accessed like a queue (first in - first out) so you can draw menus and dialogs on top of each other, while the last UI element (on top of the stack) keeps the control of the input-update-draw cycle.

In the menu stack we also have some functionalities that may not be included in a standard stack, like a "clear" function, which allows us to completely clean the stack: this can prove useful when we are accessing the main game, since we may not want to render the menu "below" the main game, wasting precious CPU cycles.

{{placeholder}}

<!-- TODO: Make a structure with a menu stack with "UI Elements" that can refer to the stack itself (for the "back" action for instance), add diagram and code -->

Particle Systems
-----------------

<!-- TODO: Talk about particle systems, particles and particle emitters -->

{{placeholder}}

### Particles

{{placeholder}}

<!-- TODO: What is a particle, what does it do, etc... -->

### Generators

{{placeholder}}

<!-- TODO: What is a particle generator -->

### Emitters

{{placeholder}}

<!-- TODO: What is a particle emitter, how does it differ from a generator -->

### Updaters

{{placeholder}}

<!-- TODO: What is a particle updater, how it works, etc... -->

### Force Application

{{placeholder}}

<!-- TODO: Talk about applying forces to particles -->

Timers
------

Timers are an essential component in many game mechanics: [Coyote Time](#coyote_time) and [Jump Buffering](#jump_buffering) are two prime examples of timer-based mechanics.

If we want to execute a function every few seconds we need timers.

If we need to cap how many bullets we can shoot from a weapon, guess what? Timers.

Making a timer is not as complicated as it may seem, we need:

- A variable that keeps track of the time passed;
- A variable that keeps track of how much time needs to pass before the function gets executed;
- A pointer to said function;
- A boolean to track whether the timer is active or not;
- A boolean to decide whether the timer should be "one shot" or "continuous".

```{src='containers/timer' caption='A simple timer class'}
```

### Accounting for "leftover time"

This timer is a nice and simple solution, but it has a small flaw: when the timer is set to execute continuously and the function is executed, it doesn't account for "leftover time". This may be easier to understand with an example.

Let's imagine that we have a timer that shoots a bullet every quarter of a second (250ms), our game is running at a steady 30fps (which means each frame takes 33.33ms), our timer's internal counter will behave like this:

- **Frame 1:** 250 - 33.33 = 216.67
- **Frame 2:** 216.67 - 33.33 = 183.34
- ...
- **Frame 7:** 50.20 - 33.33 = 16.87
- **Frame 8:** 16.87 - 33.33 = -16.46 (Trigger the function and reset the timer)
- **Frame 9:** 250 - 33.33 = 216.67

Our timer doesn't account for the over 16ms leftover that we had between frame 8 and frame 9, thus our timer will be imprecise. This may seem an easy fix at first glance, but it is not.

#### A naive solution

The first solution that would come to mind would be substituting the timer variable assignment with a sum, thus frames 7,8 and 9 would look like this:

- **Frame 7:** 50.20 - 33.33 = 16.87
- **Frame 8:** 16.87 - 33.33 = -16.46 (Trigger the function and reset the timer, by adding 250)
- **Frame 9:** 233.54 - 33.33 = 200.21

Here is the code:

```{src='containers/timer_naive' caption='A naive approach to account for leftover time'}
```

But what happens if we have a sudden lag spike, longer than the timer itself?

On **Frame 7** we have 50.20 - 500 = -449.8, due to a Lag spike: last frame took a lot longer to process, we have to execute our function and reset the timer, adding 250.

On **Frame 8** we have -199.8 - 33.33 = -233.13 : The timer is trying to catch up to the lag spike, since the trigger condition is still happening, we execute the function again and add 250 to the timer.

On **Frame 9** we have 16.87 - 33.33 = -16.46 We've almost caught up with the lag spike, but we need to execute the function a third time and add 250 to our timer.

**Frame 10** behaves normally with 233.54 - 33.33 = 200.21.

Due to the lag spike the function gets executed three times, which is the technically correct way to "catch up" with the number of times the timer *should have* triggered. But this may be an undesirable side effect.

If our game is already slowing down, executing even more functions won't help, so a better approach would definitely be avoiding calling functions more than we strictly need.

#### A different approach

To avoid this "catching up", there are many ways, I'm going to write two of them in this book. The first is quite simple: we add the set timer in a loop until we reach a value higher than zero.

```{src='containers/timer_leftover_1' caption='A possible solution to account for leftover time'}
```

This approach has a very minor issue: we are using a loop, so the further we stray away from zero, the more times we will have to add. A second approach would be calculating a "multiplier" and directly apply that to the added value, thus avoiding a loop.

```{src='containers/timer_leftover_2' caption='Another possible solution to account for leftover time'}
```

This second approach has an issue too: we will need to calculate the ceiling of a value, which may require a bit more CPU time (although most modern CPUs don't require more than a single cycle to do so).

Both approaches are valid and for longer timers even the "naive" approach is valid and fast. The choice is up to your personal taste and sensibility.

Inbetweening
--------

Inbetweening, also known as "tweening", is a method that allows to "smear" a value over time, this is usually done with animations, where you set the beginning and end position of a certain object, as well as the time the movement should take, and let the program take care of the animation.

This is particularly useful in animating *UI*~[g]~ objects, to give a more refined feel to the game.

Here we will present some simple tweenings that can be programmed, and explain them.

Let's start with a *linear* tweening, usually the following function is used:

```{src='containers/tween_linear' caption='Linear Tweening'}
```

Let's explain the variables used:

- **time**: The current time of the tween. This can be any unit (frames, seconds, steps, ...), as long as it is the same unit as the "duration" variable;
- **begin**: represents the beginning value of the property being inbetweened;
- **change**: represents the change between the beginning and destination value of the property;
- **duration**: represents the duration of the tween.

Note that the measure (time / duration) represents the "percentage of completion" of the tweening.

In some cases a Linear tweening is not enough, that's where *easing* comes into play.

Before introducing easing let's analyze the function again, if you try plugging in some data into the function, you will find that there is always going to be:

$$ (change\ in\ property) \cdot (factor) + (beginning\ value)$$

So we can use our function substituting `begin` with 0 and `change` with 1 to calculate `factor` and have a code similar to this one:

```{src='containers/way_to_easing' caption='Example of a simple easing function'}
```

With linear tweening, the function degenerates to $\frac{time}{duration}$, but now we can replace our linear tween with the following function:

```{src='containers/easeIn' caption='Ease-in'}
```

By changing the `power` parameter, we change the behaviour of the easing, making the movement slower at the beginning and pick up the pace more and more, until the destination is reached. This is called a "ease-in".

For an "ease-out", where the animation starts fast and slows down towards the end, we use the following function instead:

```{src='containers/easeOut' caption='Ease-out'}
```

With some calculations, and if statements on the time passed, you can combine the two and get an "ease-in-out" function.

```{src='containers/easeInOut' caption='Ease-in-out'}
```

Obviously these functions have an issue: they don't clamp the value between 0 and 1, that will have to be done in the movement function or by adding a check, or using some math, for instance using `min(calculated_value, 1)`.

```{src='containers/clamping' caption='Clamping values'}
```

In that case, calling the clamping function with values `0` and `1` would solve the issue.

A think that many people tend to forget, but that is really important is that you can tween any property of any entity or object in your game, for instance:

- The position of a UI item;
- The width of a health bar;
- The rotation of an on-screen map or compass;
- The colour of the sky while doing a day-to-night transition.

In short: any numeric value that can transition "smoothly" between two values in a certain amount of time can be tweened.

Chaining
--------

{{placeholder}}

<!-- TODO: An abstraction that allows to chain operations one after another, usually said operations are timed or related to tweening -->
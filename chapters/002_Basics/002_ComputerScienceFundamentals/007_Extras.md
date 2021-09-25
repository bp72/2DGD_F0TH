Data Redundancy
---------------

When dealing with certain structures, there are operations that are inherently complex to do: let's take for example counting the elements in a list:

```{src='computer_science/redundancy_1' caption='Counting the elements in a list'}
```

It's easy to see that an algorithm like this has a $\Theta(n)$ complexity, which may not be ideal for an operation as common as finding the length of a list.

This is where data redundancy comes into play: the length of a list is an intrinsic property of the list itself, so why not save it inside the "head" of our structure?

This will obviously require a bit more work in all the methods that will change the number of elements inside the list, since we need to keep the "length" property in sync with the actual length of the list, but in exchange we can count the elements in a list by doing a simple lookup.

Let's see an example implementation:

```{src='computer_science/redundancy_2' caption='Counting the elements in a list with data redundancy'}
```

:::: pitfall ::::
It is extremely important that we keep our "redundant properties" synchronised with the actual state of our objects, even when exceptions are raised. Not doing so will create bugs.
:::::::::::::::::

Let's consider another example: we have a standard linked list, like the one that follows:

![Singly-Linked List has no redundancy](./images/computer_science/linked_list_no_redundancy.svg){width=60%}

Our "pointer" is pointing the node containing the number "5", and now we want to know the value of the node that precedes it. To do that we need to start from the head, saving in a temporary variable our nodes, until we find the node pointed by our "pointer".

```{src='computer_science/redundancy_3' caption='Finding the previous element in a singly linked list'}
```

This operation has $O(n)$ complexity, which is not great. If we wanted to print a list in reverse with such technique, the situation would be even worse.

Doubly-linked lists are another example of data redundancy. We are saving the content of the "previous" node, so that we can do a simple lookup with complexity $O(1)$ and easily (and efficiently) do our "reverse printing".

![A doubly linked list is an example of redundancy](./images/computer_science/doubly_linked_list_redundancy.svg){width=60%}

Introduction to Multi-Tasking
------------------------------

When it comes to humans, we are used to have everything at our disposal immediately, but when it comes to computers, each processing unit (CPU) is usually able to perform only one task at a time.

To allow for multi-tasking (doing many activities at once), the CPU switches between tasks at high speeds, giving us the illusion that many things are happening at once. There are many methods to ensure multi-tasking without *process starvation*~[g]~, the most used is *pre-emption*~[g]~ where there are forced context switches between processes, to avoid one hogging the CPU.

### Coroutines

If you search for the word "coroutine" online, you will find a lot of extremely convoluted explanations involving the knowledge of the difference between *preemptive*~[g]~ and *non-preemptive* multitasking, subroutines, threads and lots more. Let's try to make sense of this.

First of all, coroutines are computer programs can run in multitasking (so it can run separated from our main game loop) which are used in non-preemptive multitasking. Differently from the preemptive style defined in the glossary, in non preemptive multitasking the operating system never forces a context switch, but it's the coroutine's job to **yield** the control over to another function.

Instead of "fighting for resources", coroutines politely free the processor and give control of it to something else (could be the caller or another coroutine), this form of multitasking is often called **cooperative multitasking**.

A particularly interesting point of coroutines is the fact that their execution can be "suspended" and "resumed" without losing its internal state. Coroutines are using in more advanced engines (using the *Actor Model*) and in some particular situations. You may never need to use a single coroutine, or you may need to use them every day, so it's worth knowing at least what they are.

Introduction to Multi-Threading {#multithreading}
------------------------------

<!-- TODO: How to implement multi-threading via mutex, immutable objects and atomic operations -->

When it comes to games and software, we usually think of it as a single line of execution, branching to (not really) infinite possibilities; but when it comes to games, we may need to dip our toes into the world of multi-threaded applications.

### What is Multi-Threading

Multi-Threading means that multiple threads exist in the context of a single process, each thread has an independent line of execution but all the threads share the process resources.

In a game, we have the "Game Process", which can contain different threads, like:

- World Update Thread
- Rendering Thread
- Loading Thread
- ...

Multi-Threading is also useful when we want to perform concurrent execution of activities.

### Why Multi-Threading?

Many people think of Multi-Threading as "parallel execution" of tasks that leads to faster performance. That is not always the case. Sometimes Multi-Threading is used to simplify data sharing between flows of execution, other times threads guarantee lower latency, other times again we may *need* threads to get things working at all.

For instance let's take a loading screen: in a single-threaded application, we are endlessly looping in the input-update-draw cycle, but what if the "update" part of the cycle is used to load resources from a slow storage media like a Hard Disk or even worse, a disk drive?

The update function will keep running until all the resources are loaded, the game loop is stuck and no drawing will be executed until the loading has finished. The game is essentially hung, frozen and your operating system may even ask you to terminate it. In this case we need the main game loop to keep going, while something else takes care of loading the resources.

### Thread Safety

Threads are concurrent execution are powerful tools in our "programmer's toolbox", but as with all powers, it has its own drawbacks.

#### Race conditions

Imagine a simple situation like the following: we have two threads and one shared variable.

![Two threads and a shared variable](./images/computer_science/MultiThreading1.svg){width=60%}

Both threads are very simple in their execution: they read the value of our variable, add 1 and then write the result in the same variable.

This seems simple enough for us humans, but there is a situation that can be really harmful: let's see, in the following example each thread will be executed only once. So the final result, given the example, should be "3".

First of all, let's say Thread 1 starts its execution and reads the variable value.

![Thread 1 reads the variable](./images/computer_science/MultiThreading2.svg){width=60%}

Now, while Thread 1 is calculating the result, Thread 2 (which is totally unrelated to Thread 1) starts its execution and reads the variable.

![While Thread 1 is working, Thread 2 reads the variable](./images/computer_science/MultiThreading3.svg){width=60%}

Now Thread 1 is finishing its calculation and writes the result into the variable.

![Thread 1 writes the variable](./images/computer_science/MultiThreading4.svg){width=60%}

After That, Thread 2 finishes its calculation too, and writes the result into the variable too.

![Thread 2 writes the variable](./images/computer_science/MultiThreading5.svg){width=60%}

Something is not right, the result should be "3", but it's "2" instead.

![Both Threads Terminated](./images/computer_science/MultiThreading6.svg){width=60%}

We just experienced what is called a **"race condition"**: there is no real order in accessing the shared variable, so things get messy and the result is not deterministic. We don't have any guarantee that the result will be right all the time (or wrong all the time either).

#### Critical Regions

Critical Regions (sometimes called "Critical Sections") are those pieces of code where a shared resource is used, and as such it can lead to erroneous or unexpected behaviours. Such sections must be protected from concurrent access, which means only one process or thread can access them at one given time.

### Ensuring determinism

Let's take a look at how to implement multi-threading in a safe way, allowing our game to perform better without non-deterministic behaviours. There are other implementation approaches (like thread-local storage and re-entrancy) but we will take a look at the most common here.

#### Immutable Objects

The easiest way to implement thread-safety is to make the shared data immutable. This way the data can only be read (and not changed) and we completely remove the risk of having it changed by another thread. This is an approach used in many languages (like Python and Java) when it comes to strings. In those languages strings are immutable, and "mutable operations" only return a *new string* instead of modifying the existent one.

#### Mutex

Mutex (Short for **mut**ual **ex**clusion) means that the access to the shared data is serialised in a way that only one thread can read or write to such data at any given time. Mutual exclusion can be achieved via algorithms (be careful of *out of order execution*~[g]~), via hardware or using "software mutex devices" like:

- Locks (known also as *mutexes*)
- Semaphores
- Monitors
- Readers-Writer locks
- Recursive Locks
- ...

Usually these multi-threaded functionalities are part of the programming language used, or available via libraries.

#### Atomic Operations

{{placeholder}}
<!-- TODO: Talk about atomic operations -->
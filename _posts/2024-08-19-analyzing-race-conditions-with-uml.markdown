---
layout: post
title:  "Analyzing Race Conditions With UML Sequence Diagrams"
date:   2024-08-19 12:00:47 +0300
categories: software_design
---

## Concurrency Is Hard

Sooner or later, every software engineer faces concurrent code execution.

Where there is concurrency, there are race conditions.

Debugging race conditions can be fun at times, but it's mostly annoying and
painful due to their inconsistent nature.

> The best way to avoid debugging race conditions is to eliminate them at the
> design stage.

In this post, I will show you a simple trick that will help you expose race
conditions using UML sequence diagrams.

## Example: Incrementing Non-Atomic Value From Several Threads

Let's take a look at the following code example that might come up in a job
interview when you're asked about multithreading.

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <chrono>


int main() {
    int counter{0};

    std::vector<std::thread> threads{};

    for (size_t i = 0; i < 2; i++) {
        threads.emplace_back([&counter] {
            for (int i = 0; i < 100000; i++)
            {
                counter++;
            }
        });
    }

    for (auto& thread : threads) {
        thread.join();
    }

    std::cout << "counter value is " << counter << std::endl;


    return 0;
}
```

In this program, two threads are spawned, and each one increments a non-atomic
counter in a loop. The result ends up somewhere between 100,000 and 200,000.

This behavior is caused by a race condition between the threads because
`counter++` is not an atomic operation in the general case.

Now, let's expose this race condition.

## UML it!

### Initial Representation

The UML sequence diagram of the code above might look like this:

```mermaid!
sequenceDiagram
    participant Threads
    participant Counter
par
    loop 1000 times
        Threads ->> Counter : fetch
        Counter -->> Threads : value
        note over Threads : increment
        Threads ->> Counter : store
    end
end
```

Taking into account that the increment isn't atomic in our case, we split it
into several operations in the diagram.

### Sequence "Unrolling"

The diagram above is an accurate enough representation of what is happening in
the code, but to analyze potential timing issues, we need to modify it.

Now, we will represent the sequence in an "unrolled" view:

```mermaid!
sequenceDiagram
    participant Thread1
    participant Counter
    participant Thread2
par
    loop 1000 times
        Thread1 ->> Counter : fetch
        Counter -->> Thread1 : value
        note over Thread1 : increment
        Thread1 ->> Counter : store
    end
and
    loop 1000 times
        Thread2 ->> Counter : fetch
        Counter -->> Thread2 : value
        note over Thread2 : increment
        Thread2 ->> Counter : store
    end
end
```

now we remove `par` section:

```mermaid!
sequenceDiagram
    participant Thread1
    participant Counter
    participant Thread2
loop 1000 times
    Thread1 ->> Counter : fetch
    Counter -->> Thread1 : value
    note over Thread1 : increment
    Thread1 ->> Counter : store
end
loop 1000 times
    Thread2 ->> Counter : fetch
    Counter -->> Thread2 : value
    note over Thread2 : increment
    Thread2 ->> Counter : store
end
```

then we unroll the `loop`:

```mermaid!
sequenceDiagram
    participant Thread1
    participant Counter
    participant Thread2

Thread1 ->> Counter : fetch
Counter -->> Thread1 : value
note over Thread1 : increment
Thread1 ->> Counter : store

Thread2 ->> Counter : fetch
Counter -->> Thread2 : value
note over Thread2 : increment
Thread2 ->> Counter : store
```

and the final step is starting moving arrows to see which combinations may occur:

```mermaid!
sequenceDiagram
    participant Thread1
    participant Counter
    participant Thread2

Thread1 ->> Counter : fetch
Counter -->> Thread1 : value
Thread2 ->> Counter : fetch
Counter -->> Thread2 : value
note over Thread1 : increment
note over Thread2 : increment
Thread1 ->> Counter : store
Thread2 ->> Counter : store
```

We can clearly see that there is a case where the same value is read by two
threads, and each one increments the same value before storing it.

I've represented only one case here as an example, but all possible cases
should be analyzed when designing software.

## Conclusion

As you can see, the "unrolling" is very straightforward, easily implemented,
and doesn't require special skills.

> We can save hours of design time and spend months fixing the implementation.

Yes, it may be time-consuming, but the benefit of addressing it during the
design stage is the time saved by not having to debug inconsistent behavior
later and the reduced cost of implementing a reliable solution.

## Happy Designing!

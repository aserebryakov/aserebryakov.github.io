---
layout: post
title:  "Analysing Race Conditions With UML Sequence Diagrams"
date:   2025-08-11 12:00:47 +0300
categories: software_design
---

## Concurrency Is Hard

Earlier or later each software engineer faces concurrent code execution.

When there is a concurrency there are race conditions.

Debugging race conditions is sometimes fun, but mostly annoying and painful
because of their inconsistent nature.

> Best way to avoid debugging race conditions - eliminate them on design stage

In this post I will show you a simple trick that will help you to expose them
using UML sequence diagrams.

## Example: Incrementing Non-Atomic Value From Several Threads

Let's take a look on the following code example that may appear on job interview
when you're asked about multithreading.


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

In this program there two threads are spawned and each one increments
non-atomic counter in a loop. The result is somewhere between 100000 and
200000.

It is caused by race condition between threads because `counter++` isn't atomic
operation in general case.

Now we will expose this race condition.

## UML it!

### Initial Representation

Let's represent the code in UML sequence diagram. Taking into account that
increment isn't atomic in our case we split it into several opetations on
diagram.

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

### Sequence "Unrolling"

The diagram above is a proper way to represent what is happening in the code,
but in order to analyze propable timing issues we need to change it.

Now we will represent the sequence in "unrolled" view:

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

Now let's remove `par` section

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

Then we unroll the `loop`.

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

And the final step is starting moving arrows to see which combinations may take
place.

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

We can clearly see that there is a case when the same value is read by two
threads and each one increments the same value before storing.

Here I represented only one case just for example, but all cases should be
analyzed when you're designing software.


## Conclusion

As you can see, the "unrolling" is a very straight forward that can be easily
implemented and doesn't require special skills.

> We can save hours of design time by spending months fixing the implementation

Yes, it may be time consuming, but the profit of making it on design stages is
in the time that will not be spent on trying to debug inconsistant behavior and
and cost of reliable solution implementation.


## Happy Designing!

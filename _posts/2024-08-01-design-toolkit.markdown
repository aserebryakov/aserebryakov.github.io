---
layout: post
title:  "My Personal Software Design Toolkit"
date:   2024-08-01 12:00:47 +0300
categories: jekyll update
---

## TL;DR

1. Store your design documentation with the code.
2. Use Markdown for text.
3. Use [PlantUML](https://plantuml.com/) or [Mermaid](https://mermaid.js.org/)
   for your diagrams (choose the one that can be rendered as part of Markdown
   in your environment).

But if you want to know why, read further.

## Single Source of Truth

The main idea behind the toolkit is:

> All documentation should be in one place - the project repository.

I used different tools to create software design documents:

 - Pen and paper ;)
 - MS Word
 - Confluence

And when you finish your work on the design, you need to answer the following questions:

 - Where to store your design document
 - How to track changes in the design document

### Where to store your design document?

There are a number of answers to these questions:

 - Network drive
 - Cloud storage
 - Document management system

All these answers are valid (Confluence itself is already a storage), but all
of them require some kind of deviation from your regular workflow.

If you're a "Onetaskman" like me, it may be annoying, especially when you
need to make a small change after a review or when you find some detail that
needs to be updated.


### How to track changes in a design document?

If it's an MS Word document, I use the "Track Changes" feature that clearly
represents added, removed, or updated text. My usual practice in this case was
to keep the updates list in the document and not turn it off, so the latest
changes always stay highlighted.

So, when you update the design document, you send a copy with the latest
changes highlighted and then store the result in your storage.


### One Place to Rule Them All

As you see, most of the options require deviating from your regular workflow
and remembering things that you're not frequently using.

But we have a code repository. It is the most robust part of all systems simply
because it is essential for developing a product.

Many other things can be offline or have issues that prevent you from adding or
updating the document. I think we all know how infrastructure migrations can
become a disaster when no one has access to anything, but even in this case,
code access will be restored first.

#### Design Reviews

Design reviews can be conducted by reviewing a Pull/Merge Request and pushing
documentation changes to the main development branch.

You automatically know who reviewed this design, what was changed, and by the
presence of the document in the main development branch, that this design was
accepted.


## Toolkit components

When we decided to use the repository as documentation storage, we needed to
choose the tools we would use.


### Text

Obviously, [Markdown](https://daringfireball.net/projects/markdown/) is the
answer here.

It provides all the necessary tools for our design text part, it is rendered by
everything, and it can be read without rendering.

Of course, change tracking is trivial.


### Diagrams

I have two options in my kit for drawing diagrams:

- PlantUML
- Mermaid

Both are text-based, which makes them easy to update while working on new
designs. For example, if you need to add a new participant to a sequence
diagram, you can do so directly without needing to manually adjust all the
arrows, which can be quite cumbersome in simple graphical editors.

Additionally, IDEs such as `CLion` can propose updates to these files when you
use the `refactor` functionality (e.g., when you change a class name). This
feature increases the likelihood that documentation remains up-to-date when
changes are made.

There are plugins for IDEs that allow you to integrate these tools into your
workflow without needing additional software.


#### PlantUML

[PlantUML](https://plantuml.com/) sequence diagram for a taxi ordering looks
like:

```plantuml
actor Client
participant Dispatcher
participant TaxiDrivers

Client -> Dispatcher : "I need a taxi"
Client <- Dispatcher : "Please wait"

loop for each driver
    Dispatcher -> TaxiDrivers : "Are you free?"
    alt driver is free
        Dispatcher <- TaxiDrivers : "Yes, I am"
        Dispatcher -> TaxiDrivers : "Here is the client's address"
        Dispatcher <- TaxiDrivers : "On my way"
    else
        TaxiDrivers -> Dispatcher : "No, I am with a client"
    end

    alt free driver is found
        note over Dispatcher : stop querying drivers
    end
end

alt free driver is found
    Client <- Dispatcher : "Please wait for your taxi"
else
    Client <- Dispatcher : "Unfortunately, we don't have free cars now"     
end
```

And the rendered result may look like:

```plantuml!
actor Client
participant Dispatcher
participant TaxiDrivers

Client -> Dispatcher : "I need a taxi"
Client <- Dispatcher : "Please wait"

loop for each driver
    Dispatcher -> TaxiDrivers : "Are you free?"
    alt driver is free
        Dispatcher <- TaxiDrivers : "Yes, I am"
        Dispatcher -> TaxiDrivers : "Here is the client's address"
        Dispatcher <- TaxiDrivers : "On my way"
    else
        TaxiDrivers -> Dispatcher : "No, I am with a client"
    end

    alt free driver is found
        note over Dispatcher : stop searching for driver
    end
end

alt free driver is found
    Client <- Dispatcher : "Please wait for your taxi"
else
    Client <- Dispatcher : "Unfortunately, we don't have free cars now"     
end
```

The issue I faced was that PlantUML wasn't natively supported on the Git
platforms used at work. As a result, I had to render diagrams myself and then
store the rendered images in the repository, which isn't ideal, especially with
limited repository size.

On the other hand, PlantUML currently offers better control over diagrams. For
example, it is possible to specify the placement of classes relative to each
other, such as:


```plantuml
Center -up-> Up
Center -down-> Down
Center -left-> Left
Center -right-> Right
```
Is rendered as:

```plantuml!
Center -up-> Up
Center -down-> Down
Center -left-> Left
Center -right-> Right
```

Instead of default:

```plantuml!
class Center
class Up
class Down
class Left
class Right

Center -> Up
Center -> Down
Center -> Left
Center -> Right
```

Another disadvantage is that PlantUML doesn't specify the type of diagram
intended; instead, the result is determined by the content.

#### Mermaid

The same diagram of taxi ordering in [Mermaid](https://mermaid.js.org/):

```mermaid
sequenceDiagram
    actor Client
    participant Dispatcher
    participant TaxiDrivers

Client ->> Dispatcher: "I need a taxi"
Dispatcher ->> Client: "Please wait"

loop for each driver
    Dispatcher ->> TaxiDrivers: "Are you free?"
    alt driver is free
        TaxiDrivers ->> Dispatcher: "Yes, I am"
        Dispatcher ->> TaxiDrivers: "Here is the client's address"
        TaxiDrivers ->> Dispatcher: "On my way"
    else
        TaxiDrivers -> Dispatcher: "No, I am with a client"
    end

    alt free driver is found
        note over Dispatcher: stop searching for driver
    end
end

alt free driver is found
    Dispatcher ->> Client: "Please wait for your taxi"
else
    Dispatcher ->> Client: "Unfortunately, we don't have free cars now"
end
```

That is rendered as:

```mermaid!
sequenceDiagram
    actor Client
    participant Dispatcher
    participant TaxiDrivers

Client ->> Dispatcher: "I need a taxi"
Dispatcher ->> Client: "Please wait"

loop for each driver
    Dispatcher ->> TaxiDrivers: "Are you free?"
    alt driver is free
        TaxiDrivers ->> Dispatcher: "Yes, I am"
        Dispatcher ->> TaxiDrivers: "Here is the client's address"
        TaxiDrivers ->> Dispatcher: "On my way"
    else
        TaxiDrivers -> Dispatcher: "No, I am with a client"
    end

    alt free driver is found
        note over Dispatcher: stop searching for driver
    end
end

alt free driver is found
    Dispatcher ->> Client: "Please wait for your taxi"
else
    Dispatcher ->> Client: "Unfortunately, we don't have free cars now"
end
```

The main advantage of Mermaid that I personally experienced was its
out-of-the-box support in GitLab at the time I was using it. This made it my
first choice during that period.

As for disadvantages, Mermaid currently offers fewer options than PlantUML.

## Happy Designing!

---
layout: post
title:  "My Personal Software Design Toolkit"
date:   2024-08-02 12:00:47 +0300
categories: jekyll update
---

# Single Source of Truth

The main idea behind the toolkit is:

> All documentation should be in one place - project repository.

Let me explain it a little.

I used different tools to create software design documents:

 - Pen and paper
 - MS Word
 - Confluence

And when you finish your work on design, you need to answer following questions:
 - Where to store your design document
 - How to track changes in design document

## Where to store your design document?

There is a number of answers for these questions:

 - Network drive
 - Cloud storage
 - Document management system

All these answers are valid (Confluence itself is already a storage) but all of
them require some kind of deviation of your regular workflow.

If you're "Onetaskman" like I am, it may be annoying. Especially when you need
to do a little change after review or you found some detail that requires to be
updated.

## How to track changes in design document?

If it's a MS Word document I used "Track changes" feature that clearly
represents added/removed/updated text. My usual practice in this case was
keeping updates list in documents and do not turn it off so latest change stays
always highlighted.

So when you update design document you send a copy with latest changes
highlighted and then store result in your storage.

## One place to rule them all

As you see, the most of the options require either deviate of your regular
workflow and remember things that you're not frequently using.

But we have a code repository. It is the most robust part of all systems
just because of the fact that it is essential for developing a product.

Many other things can be offline or have any issues that won't allow you to
add/update the document. I think we're all know how infrastructure migrations
can become a disaster when no one have access to anything, but even in this
case code access will be restored in the first place.

### Design reviews

Design reviews can be made by reviewing a Pull/Merge Request and pushing
documentation changes to a main developing branch.

You automatically know who was reviewing this design, what was changed, and by
the fact of the presence of the document in main developing branch that 
this design was accepted.

# Toolkit components

When we decided that to use the repository as a documentation storage we need
choose our tools that we will use.

## Text

Obviously Markdown is the answer here. 

It provides all necessary tools for our design text part, it is rendered by
everything and can be read without rendering.

Of course change tracking is trivial.

## Diagrams

I have two options in my kit to draw diagrams:

 - PlantUML
 - Mermaid

Both of them are text based that makes them easy to update while working on new
design because. If you need to add e.g. a new participant to a sequence diagram,
you can put it right in the place without need to fix all arrows manually that
is really painful in simple graphical editors.

In addition to it IDEs such as `CLion` can propose to update these files when
you're using `refactor` functionality (when you change a class name for example).
This fact increases chances of documentation to be up to date when changes are
made.

There are plugins for IDEs that allow you to integrate them into your workflow
without a need to use some other software.

Of course both PlantUML and Mermaid had advantages and disadvantages while I
was working with them, so let's dig a bit deeper.

### Plantuml

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

The issue that I faced was that PlantUML wasn't natively supported in Git
platforms that were used at work, thus I had to render diagrams on my own and
then store rendered images in repo that isn't that good especially when you
have limited repo size.

On the other hand PlantUML at the moment gives better control over the diagram
e.g. it is possible to specify where classes will be placed relatively to each
other like:

```plantuml
Center -up-> Up
Center -down-> Down
Center -left-> Left
Center -right-> Right
```
Renders to:

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

Another disadvantage is that PlantUML doesn't specify what type of diagram is
intended, and the result is defined by the content.

### Mermaid

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

That is rendered to:

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

The main advantage of Mermaid that I personally faced was that it had out of
the box support in GitLab by the time I was using it. That made it my first
choice for this period.

As for disadvantages, it has less options at the moment than PlantUML.

# TL;DR

  1. Store your design documentation with the code
  2. Use Markdown for a text
  3. Use PlantUML or Mermaid for your diagrams (choose the one that can be
     rendered as part of Markdown in your environment)

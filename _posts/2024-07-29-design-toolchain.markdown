---
layout: post
title:  "My Personal Software Design Toolkit"
date:   2025-07-29 12:00:47 +0300
categories: jekyll update
---

# Single Source of Truth

The main idea behind the toolkit is:

> All documentation should be in one place - project repository.

Let me explain it a little.

In my experience I used different tools to create software design documents:

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

If you're "Onetaskman" like I am, it may be a little bit annoying. Especially
when you need to do a little change after review or you found some detail that
requires to be updated.

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

 - plantuml
 - mermaid

Both of them are text based so you 
And both of them have advantages and disadvantages, so let's dig a bit deeper.

### Plantuml



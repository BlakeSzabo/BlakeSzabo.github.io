---
title: Emergent Computers 
date: 2024-08-09 21:00:00+1000
categories: [Computer Science]
tags: [Computer Science]
author: blake_szabo
description: So you've accidentally calculated something; now what? 
---

Let me ask you a question: What was the first computer? I think it's
an interesting question that likely boils down to what you _really_ define
as a computer. The word "computer" was originally used as a job title for us mere
mortals dating back as far as the 1600s. Furthermore, there were also
mechanical devices that would perform calculation, which were effectively clockwork
mechanisms that would leverage gear ratios to perform multiplication. Early electronic
computers were essentially iterations on these early mechanical devices in that they
were extremely specialized and built for a specific purpose. I would say though,
that a defining quality of the modern computer is that it is programmable. It isn't
built for a single purpose but rather satisfies the whole spectrum of computable
problems; they're **Turing-complete**.


What does it mean that modern computers are Turing-complete? To be exact, it means
that they are capable of fully simulating any Turing machine. Okay, what does that mean? Well, we _could_ go down
that rabbit hole and explore the world of computation theory, but I will leave
that story for another day. For now I think it's okay just to think of a "Turing machine"
as just a modern computer for the sake of simplicity. The first physical construction
we made that was Turing-complete was the ENIAC in 1946. You may have also heard
that most programming languages are Turing-complete, and that's true! Some may even
restrict the term "programming" language only to languages that satisfy this condition.
Essentially, this means that most programming languages are capable of simulating
a computer inside the program they are running. However, it's not always by design
that something is Turing-complete. Take, for example: Minecraft.


# Minecraft

Minecraft is a block-based sandbox game that gives the player freedom to build
almost whatever their mind desires. It also has a little system called redstone.
Redstone is a system built on simple rules: you can place it on the ground, and it will
connect with adjacent redstone to create a "wire" of sorts. You can then power
the redstone either with an input like a button or switch, or you can place a
redstone "torch", which will essentially power all the redstone it is connected to.
Finally, out of the basic components, there's the repeater which essentially functions
as a "delay" to the signal passed in. When it receives power from one end, it will
pass on that power to the other side after a variable delay.


One thing to mention about the torches is that if the block that they are connected to
becomes powered, then the torch will be disabled. This is the critical last piece of the
puzzle that allows redstone to become Turing-complete. This has spurred many people 
to create their own functional CPUs in their Minecraft worlds and share them online.
You can find many videos showing off the giant contraptions on YouTube. Some even
take it a step further and implement a large display and implement a smaller, simpler
version of Minecraft within Minecraft.

Minecraft is an expansive and very open-ended game, so while it is really cool
that you can create something like this within the bounds of the game, I'm not
actually that surprised by that fact. Why? Well it's actually quite "simple" to
create a Turing-complete system.


# Oh God, It's Everywhere

Microsoft Excel, Conway's Game of Life, the type system of TypeScript itself, and
even just the single "mov" instruction from the x86 instruction set are all Turing-complete.
These are just a few examples, but I hope it illustrates the point that many things
from simple to complex share this quality. Even the Turing Machine itself follows a
relatively simple set of rules. My favourite example of this, by far, is the trading
card game Magic: the Gathering (MtG). MtG is far from simple, but it is also, in
my opinion, uniquely special in that, by the nature of the game, it necessarily involves
another poor, poor soul in your cruel quest for computation. The simulation itself, once
it starts, involves only forced moves in the game and anything short of surrender will not free
your opponent.

It's amazing to me that there are so many examples, but it more likely illustrates
that what computers are actually doing is fundamentally quite simple. A lot of
simple operations compounded upon each other, built up over many decades of human
effort, have lead us to where we are today. It really is just data encoded as 0s and 1s
at the end of the day.

# Are Computers Inevitable?

Part of me wonders, were we always destined to have computers like this, built on
base-2 and Boolean logic? There have been computers that use base-10, the
aforementioned ENIAC being one of them, but they end up being difficult to
maintain due to physical constraints for little payoff proportionally.
Maybe there are only so many examples because it's the only system that we actively
look for. If aliens were to create their own computers, would they be doing
fundamentally the same thing? I have a lot of questions, and I'll answer
exactly none of them here. Sorry to disappoint.

# Modules Should Be Deep

One of the most important techniques for managing software complexity is to design systems so that developers only need to face a small fraction of the overall complexity at any given time. This approach is called *modular design*.



## Modular design

The goal of modular design is to minimize the dependencies between modules.

In order to manage dependencies, we think of each module in two parts: an *interface* and an *implementation*.



## What's in an interface

The interface to a module contains two kinds of information: formal and informal.

The formal parts of an interface are specified explicitly in the code, and some of these can be checked for correctness by the programming language

The informal parts of an interface include its high-level behavior, such as the fact that a function deletes the file named by one of its arguments.

One of the benefits of a clearly specified interface is that it indicates exactly what developers need to know in order to use the associated module. This helps to eliminate the “unknown unknowns” problem.



## Abstraction

An abstraction is a simplified view of an entity, which omits unimportant details.

The interface presents a simplified view of the module’s functionality; the details of the implementation are unimportant from the standpoint of the module’s abstraction, so they are omitted from the interface.

The key to designing abstractions is to understand what is important, and to look for designs that minimize the amount of information that is important.



## Deep Modules And Shadow Modules

Module depth is a way of thinking about cost versus benefit. 

![image-20230425123225232](assets/03_Modules%20Should%20Be%20Deep/image-20230425123225232.png)

> Red flag
>
> A shallow module is one whose interface is complicated relative to the functionality it provides. Shallow modules don’t help much in the battle against complexity, because the benefit they provide (not having to learn about how they work internally) is negated by the cost of learning and using their interfaces. Small modules tend to be shallow.
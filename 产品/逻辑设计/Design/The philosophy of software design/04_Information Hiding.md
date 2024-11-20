# Information Hiding(and leakage)

## Information hiding

The most important technique for achieving deep module is information hiding. The baisc idea is that each module should enscapsulate a few piece of knowledge, which represent design decisions. The knowledge is embded in the module's implementation but doesn't appear in its interface, so it is not visible to other modules.

It reduces complexity in two ways:

1. It simplifies the interface to the module, this reduces the cognitive load on developers who use the module.
2. It makes easier to evolve the module.



## Information leakage

Information leakage occurs when a design decision is reflected in multiple modules.

> Red flag
>
> Information leakage occurs when the same knowledge is used in multiple places, such as two different classes that both understand the format of a particular type of file.



## Temporal decomposition

In temporal decomposition, the structure of a system corresponds to the **time order** in which operations will occur。

When designing modules, focus on the **knowledge that’s needed to perform each task**, not the order in which tasks occur.



## Information hiding within a class

Try to design the **private methods** within a class so that each method encapsulates some information or 
capability and hides it from the rest of the class. 

In addition, try to **minimize the number of places where each instance variable is used**.



## Conclusion

Information hiding and deep modules are closely related.

If a module hides a lot of information, that tends to increase the amount of functionality provided by the module while also reducing its interface. This makes the module deeper.Conversely, if a modlue doesn't hide much information, then either it doesn't have much functionality, or it has a complex interface; either way, the module is shallow.

When decomposing a system into modules, try not to be influenced by the order in which operations will occur at runtime; that will lead you down the path of temporal decomposition, which will result in information leakage and shallow modules. Instead, think about the different pieces of knowledge that are needed to carry out the tasks of your application, and design each module to encapsulate one or a few of those pieces of knowledge. This will produce a clean and simple design with deep modules.
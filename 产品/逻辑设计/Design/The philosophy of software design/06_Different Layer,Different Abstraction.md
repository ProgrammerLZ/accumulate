# Different Layer,Different Abstraction

Software systems are composed in layers, where higher layers use the facilities provided by lower layers.

If a system contains adjacent layers with similar abstractions, this is a red flag that suggests a problem with the class decomposition.



## Pass-through methods

A pass-through method is one that does little except invoke another method, whose signature is similar or identical to that of the calling method.

Pass-through methods are bad because they contribute no new functionality.



## When is interface duplication OK?

* Dispatcher
* Interfaces with multiple implementations



## Interface versus implementation

Another application of the “different layer, different abstraction” rule is that the interface of a class should normally be different from its implementation: the representations used internally should be different from the abstractions that appear in the interface.

The difference between interface and implemetation represents valuable functionality provided by the class.



## Conclusion

Each piece of design infrastructure added to a system, such as an interface, argument, function, class, or definition, adds complexity, since developers must learn about this element. In order for an element to provide a net gain against complexity, it must eliminate some complexity that would be present in the absence of the design element. Otherwise, you are better off implementing the system without that particular element. For example, a class can reduce complexity by encapsulating functionality so that users of the class needn’t be aware of it.

The “different layer, different abstraction” rule is just an application of this idea: if different layers have the same abstraction, such as pass-through methods or decorators, then there’s a good chance that they haven’t provided enough benefit to compensate for the additional infrastructure they represent. Similarly, pass-through arguments require each of several methods to be aware of their existence (which adds to complexity) without contributing additional functionality.
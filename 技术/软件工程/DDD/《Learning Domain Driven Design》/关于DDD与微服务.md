对于一个数字化领域的一个数字化项目来说，DDD是一个非常好用的工具箱，他其中包含了很多有用的工具（模式）。无论是在战略设计阶段还是战术设计阶段，他都有合适的工具来帮助我们完成任务。

战略设计阶段。

这个阶段要记住的是，虽然你是一个工程师，但是这个阶段要尽量避免引入技术上的概念，而是以业务分析为主。这个阶段的首要角色是业务分析师，由他们来探索整个的业务。识别是否是核心域的时候不是仅仅简单的看这个子域是否给公司带来了收益，也要看这个业务的软件方面是否真的是核心域。

**领域分析**，这个工具主要是帮助在高层把握设计方向，需要使用这个工具与业务分析师共同分析出不同的子域属于哪种类型，分析的主要根据是不同类型的子域的特性，然后根据不同子域的类型，对未来的技术决策做高层的指导。



**限界上下文**，这个工具是用来保护统一语言的无歧义性的，只要是满足了统一语言的无歧义性，这个上下文就满足了限界上下文的最低标准的要求，但是未必是一个合格的限界上下文。合格的限界上下文需要良好的设计。

子域与限界上下文之间的关系可以作为一个设计的考量因素， 理想情况下一个子域可以对应一个限界上下文，但是子域也可以对应多个限界上下文，同时多个子域也可以对应同一个限界上下文，完全取决于具体的情况。限界上下文的边界的划分没有一个统一的标准，开始的时候可能限界上下文的范围会很大，但是随着对业务的逐渐深入的了解可以一点点的从这个初始的上下文中抽离出一些合适的上下文。

需要注意的是，限界上下文之间需要进行集成（有很多集成的方法，取决于具体情况），不同的限界上下文应该由不同的团队来进行开发。虽然限界上下文很像微服务架构中的微服务，但是这里我们并没有考虑任何与微服务相关的因素，这种相似仅仅是巧合而已，历史上DDD出现的时候也并没有用它去设计微服务（不过我觉得，他们包含着相似的设计思想）。



战术设计阶段。

**架构设计**。这里先说微服务架构。在战略设计阶段的工作会指导我们的微服务的设计。微服务也不是说上来就要进行微服务，可以通过在一个单体架构中对模块进行良好的解耦，在项目的初始阶段也可以达到很好的效果，并且能够为后续将单体架构拆分成微服务架构留下良好的底子。

**复杂度**。业务复杂度+技术复杂度。活动记录、聚合状态、事件源的考量。









演进式

无论是战略设计阶段还是战术设计阶段，都不可能一次就做对，在犯错的基础之上，我们对于业务域的理解以及对于技术决策的理解都会不断地逼近于该领域的一个正确的样貌。同时，业务也是在不断地发展的，这个正确的样貌本身也是不断地变化的，我们只需要做出足够的设计，而不需要完美的设计，然后在之后的过程当中不断地进行修正。但是，需要注意的是，初始的设计要能够拥抱变化，不只是业务的变化同时也是关于重构的的变化，以能够快速的将我们的系统演进为另一个更逼近与正确的样貌的样子。
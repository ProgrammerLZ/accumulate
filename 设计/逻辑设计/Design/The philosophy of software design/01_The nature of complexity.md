# The nature of commplexity

*Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system.*

## Symptoms of complexity

* Change amplification

  A seemingly simple change requires code modifications in many different places.

* Cognitive load

  How much a developer needs to know in order to complete a task.

* Unknown unknows(**Worst**)

  It is **not** **obvious** which pieces of code must be modified to complete a task, or what information a developer must have to carry out the task successfully.

One of the most important goals of good design is for a system to be *obvious*.



## Causes of complexity

Complexity caused by two things: **dependencies and obscurity**.

Dependencies and obscurity account for the three manifestations of complexity. Dependencies lead to change amplification and high cognitive load. Obscurity creates unknown unknow and also contribute to cognitive load.



## Complexity is incremental

Complexity isn’t caused by a single catastrophic error; it accumulates in lots of small chunks. In order to slow the growth of complexity, you must adopt a “**zero tolerance**” philosophy.



## Conclusion

*The bottom line is that complexity makes it difficult and risky to modify an existing code base*.
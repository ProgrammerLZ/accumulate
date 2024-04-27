## Course Overview

1. Use UML to document architectural design.
2. Explain how architecture can be understood from multiple perspectives and through different UML diagrams.
3. Explain different software achitectures,including
   1. When and how they are used.
   2. Their advantages and disadvantages.
   3. How to represent them visually.
4. Analyze and evoluate system architectures.
5. Explain how software architecture tied to:
   1. implementation languages
   2. deployment environments
   3. the structure of the development team
   4. the needs of business and development cycle
   5. opportunities for product lines
   6. the intent of the software

Software Architecture covers how a software system is constructed at the highest level.



## Module 1:UML Architecture Diagrams

### Architecture Overview and Process

* Software architecture is the fundamental design of an entire soft system.

* A software architect has to take many factors into consideration:

  * the purpose of the system
  * the audience or users of the system
  * the qualities that are of most importance to users
  * where the system will run

* Software architecture is important, particularly for large system:

  * provide a solid basis for developers to follow

* Advantages:

  * Higher productivity for the team,
  * improved evolution for the software
  * enhanced quality in the software

* In order to develop software architecture, architects must take into account the **stakeholders** of the system. 

  * Software developers
  * PM
  * Clients
  * End Users

  

### Kruchten's 4+1 View Model 	

1. The **logical view**, which focuses on the functional requirements of a system, usually involves the objects of the system.
   * UML diagrams related to the logical view are **class diagram and state diagram**.
2. The **process view** focuses on achieving non-functional requirements. These are the requirements that specify the desired **qualities** for the system, which include quality attributes such as performance and availability.
   * UML diagrams related to the process view of a system are the **activity diagram and the sequence diagram**.
3. The **development view** describes the hierarchical software structure.
   * It is concerned with deatail of software development and what is involved to support that.
   * It extends to management details such as scheduling,budgets,and work asignment.
   * Essentially, it covers the hierarchical software structure and project management.
4. The physical view handles how elements in the logical, process, and development views must be mapped to different nodes or hardware for running the system.
   * UML diagrams related to the physical view of a system is the **deployment diagram**.

5. Scenarios align with the use cases or user tasks of a system and show how the four other views work together.

Being able to see a complex problem in many different perspectives helps make your software more versatile.



### Component Diagrams

* Activity Diagram
* Deployment Diagram
* Package Diagram
* Component Diagram



## Module 2:Architecture Styles

Explain importance software architectures, their key characteristics, and how they are used in practice, which include:

* Language-based systems
* Repository-based systems
* Layered systems
* Interpreter-based systems
* Dataflow systems
* Implicit invocation systems
* Process control systems



### Langurage-Based System




















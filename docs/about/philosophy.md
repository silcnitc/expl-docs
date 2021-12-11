---
title: 'About the project'
hide:
    - toc
---
# Philosophy

Any pedagogical compiler implementation project offered as part of an undergraduate junior level compiler design course needs to satisfy two fundamental requirements:  

1. The student must be given as much depth and detail as possible about the central concepts. That is, the project must be sufficiently non-trivial to be instructive.
  
2. The quantity of work involved must not exceed what a student is able to complete in about four months, taking also into consideration the fact that she/he would be crediting several other courses concurrently. Thus, the project must be sufficiently simple to be do-able.

The problem before the teacher is to decide on how much compromise on (1) must be done in favour of (2). The present project is our stance in this matter. We expain our choices here.

There are two aspects associated with the design and implementation of each major functional unit of a compiler. – 1) A _Policy_ and 2) An implementation _Mechanism_. As an illustration, consider dynamic memory allocation. Fixed size allocation is a policy. The most common implementation mechanism is to have a run time library that manages allocation from the the heap. As another example, consider passing of parameters to functions. Call by value/call by reference are a policies. Passing values/addresses through a run time stack is the standard implementation mechanism. Yet another example, a language may describe a dynamic method binding policy for supporting subtype polymorphism in single inheritence hierarchy. Virtual function table mechanism is the most common way to implement the policy. Of course, each of the above policies and mechanisms need more detailed and concrete specification. Detailed descriptions of the policies and mechanisms associated with various functions of a compiler are given in the project documentation.

The pedagogical strategy is to specify a programming language that has a _very simple policy_ associated with functional unit of the compiler. This allows the student to implement the policy easily, once the implementation mechanism is understood. For instance, fixed-size allocation policy is specified for dynamic memory allocation. We provide detailed tutorials for various implementation mechanisms necessary to implement the policies associated with each such functional unit of the compiler. This helps the student to learn the implementation mechanisms quickly. Once the student completes the implementation one simple policy that uses a particular mechanism, she will be in a position to visualize the implementation of more sophisticated policies associated with a particular functionality that use similar mechanisms without actually going through an implementation project.

The project is designed to be done concurrently with a theory course in Compiler Design, typically offered during the third year of an undergraduate CS curriculum. It by no means is a replacement to the theory course, but is only designed to suppliment it. The project will help in solidifying the student's grasp of compiler design theory - about how the components of a compiler can be assembled together to form the whole. The student will experience the end-to-end process of translation from a high level language source to a target executable. The road-map is designed in such a way that as far as practicable, only one new concept is introduced at a time.

It is our responsiblity to explicitly state some central concepts that the project fails to teach. Here are some key omissions:  

1. The machine architecture, virtual memory model (and the the target ABI in general) are unrealisitically simple and custom made. Consequently it is easy to generate code for the target platfform, but the platform is far from any real system. To illustrate the level of oversimplification, a memory word of the target machine is assumed to be capable of storing an arbitrary interger or a string. Floating point handling is ommitted. The executable file format is extremely simple to understand, but too restrictive for real use. (There are simplifications in the source language specification as well, but those are less dramatic.)
  
2. The project is front-end intensive, but has a thin back-end. Code, register or memory optimizations are not attempted. The focus here is to get working target code generated as easily as possible. The roadmap asks the students to directly traverse an intermediete abstract syntax tree representation of the program and generate assembly language code. This is followed by a linking phase to resolve labels in the target code. Improving the back-end code generation process would require data flow analysis (at least liveliness analysis) which is better done with the support of tools like LLVM. Going into these directions would make the project too heavy for one semester. Typically, a second course in compiler design takes up these topics anyway.
  
3. Focus has not been placed on efficient data structuring or following software engineering practices religiously. The road-map suggests a simple implementation that can be built incrementally.

The project is designed to help the student to gain knowledge, appreciation and insight into the working of compilers, but does not try to train the student in professional compiler writing. This particular pedagogical choice has been taken to spare the student from the technicalities of machine architecture, executable/object formats and the complexities of the ABIs of real systems – something that in our experience seems to drive a lot of students away from systems projects. Our hope is that once the basic material is assimilated, the student will be more confident to get involved with these details in professional life.

Feedback data collected from students who credited the course is summarized [here](../feedback/index.md).

This project is the second one in a suite of two student learning projects designed to tutor undergraduate CS juniors. The first one in the suite is an OS development project ([exposnitc.github.io](http://exposnitc.github.io)), which asks the student to implement a simple multi-tasking operating system on the same machine architecture used in the present project. The target code of the compiler implemented in the present project is designed to be executable by the OS implemented in the first project. Our hope is that by going through the projects, the student will gain a clear conceptual understanding of how the two central software systems – the OS and the compiler - work together in a computer system.



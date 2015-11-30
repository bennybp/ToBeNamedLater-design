# PROP 0015 - The Graph (rename to avoid confusion with class that implements it?)

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Dynamic snapshot of the computation       |
| Status:        | In development                            |
 

## Purpose
Particularly in the electronic structure development community, there is no such things as a typical electronic structure computation.  The purpose of the graph is to store the module call hierarchy of the computation.  Because each module stores the molecule, basis, and options it was called with, as well as any results it generates, the graph also serves as a means of organizing the information.  Checkpointing the graph is sufficient to restart the computation.  The graph evolves in a dynamic manner so that arbitrarily complex call trees can be handled in a consistent manner.  Everytime a module calls another module a node is added to the graph.  For example consider a geometry optimization at the Hartree-Fock level of theory:

![Alt text](http://g.gravizo.com/g?
  digraph G {
    aize ="4,4";
    GeomOpt [label="Geometry Optimization",shape=box];
    node [shape=box,label="Hartree-Fock"]
    GeomOpt -> HF [label="Iteration 1"];
    node [shape=box,label="Other Stuff"]
    HF -> OS;
    node [label="Hartree-Fock"]
    GeomOpt -> HF2 [label="Iteration 2"];
    node [shape=box,label="Other Stuff"]
    HF2 -> OS2;
    node [label="Hartree-Fock"]
    GeomOpt -> HF3 [label="Iteration N"];
    node [shape=box,label="Other Stuff"]
    HF3 -> OS3;
  }
)

Initially we call the Geometry Optimization module.  This then calls the Hartree-Fock module, which calls whatever it needs to in order to compute an energy and a gradient.  Based on the gradient, the Geometry Optimization module calls another Hartree-Fock module and the cycle continues until an optimized structure is found.  If one wants to plot the energy or molecular structure as a function of geometry optimization iteration number, simply looping over the Hartree-Fock nodes would accomplish this.

##Structure
Mathematically speaking, the graph is a directed, acyclic graph (DAG), which means the edges have a direction (by convention it is from parent module to the child module) and no circular module calls are allowed, i.e. there can be no control flow like:

![Alt text](http://g.gravizo.com/g?
  digraph G {
    node [shape=box]
    N1 [label="Node 1"];
    N2 [label="Node 2"];
    N1 -> N2;
    N2 -> N1;
  }
)

This flow would mean that Node 1 is both the parent and child of Node 2 and vice versa.  Since one must have created the other, this is an impossible situation.

Programatically speaking, the graph is an instance of our graph library [which in turn wraps Boost's graph library (BGL)].  The BGL is quite generic and somewhat akward to use, which is why we have wrapped it.  The decision to include a graph library stems from it being useful for at least one other important application: describing a molecule's topology.  Details pertaining to usage of the graph should consult the graph library's documentation.


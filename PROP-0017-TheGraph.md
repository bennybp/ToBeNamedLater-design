# PROP 0017 - The Graph (rename to avoid confusion with class that implements it?)

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Dynamic snapshot of the computation       |
| Status:        | In development                            |
 

## Purpose
Particularly in the electronic structure development community, there is no such things as a typical electronic structure computation.  The purpose of the graph is to store the module call hierarchy of the computation as well as the input and output of each computation.  With proper integration, checkpointing the graph will be sufficient to restart the computation.  The graph evolves in a dynamic manner so that arbitrarily complex call trees can be handled in a consistent manner.  

The graph starts with an initial node, the initial graph state (to be enumerated shortly) and the initial module to be called.  Execution then proceeds by running this module, which we now term the parent module.  During the parent module's execution, more than likely, additional modules will be called, each of which is termed a child module.  These child modules are added to the graph with an edge running from the parent to the child.  Each child is given copies of its parent's graph state, but the parent module may modify this as it sees fit, before executing the child module.  Upon running a child module, execution enters the child module, which is now considered the parent module, and the now parent module uses its current graph state to run its computations.  The now parent module may call other child modules and the cycle continues.  Upon completing, program execution returns to the current module's parent, when all modules have completed program execution terminates.

The state of the graph contains:

1. A shared pointer to a constant system (molecule plus external fields)
2. A shared pointer to a constant basis set
3. A shared pointer to an options object
4. A shared pointer to a constant reference (currently termed the wavefunction, which contains information related to the state of the system)

Additionally Boost allows us to associate one instance of a class with the overall graph.  Trivially, this is somewhat akin to say naming the graph, but the fact that it is templated and not limited to a string allows us to exploit this and to instead store:

1. The original molecule specification
2. The original basis set
3. The original options
4. The original wavefunction (usually a null object)
5. The cache that will be used by the module locator

in the graph.

##Sample Usage

To illustrate the user of the graph consider a geometry optimization at the Hartree-Fock level of theory:

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

Initially we call the Geometry Optimization module.  This then calls the Hartree-Fock module, which calls whatever it needs to in order to compute an energy and a gradient, which it returns.  Based on the returned energy and gradient, the Geometry Optimization module updates the geometry and then calls another Hartree-Fock module. This cycle continues until an optimized structure is found.  Note that each node stores the molecule it was given, so if one wants to make a movie of the molecular structure as a function of iteration number, simply looping over the Hartree-Fock nodes would accomplish this.

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

This flow would mean that Node 1 is both the parent and child of Node 2 and vice versa.  Since one must have created the other, this is an impossible situation.  This is not to say that Node 2 can not call another instance of Node 1, which would result in:
![Alt text](http://g.gravizo.com/g?
  digraph G {
    node [shape=box]
    N1 [label="Node 1"];
    N2 [label="Node 2"];
    N3 [label="Node 1, Call 2"]
    N1 -> N2;
    N2 -> N3;
  }
)
As soon as Node 2 returns, control returns to Node 1, which is why cycles cannot exist.

Programatically speaking, the graph is an instance of our graph library [which in turn wraps Boost's graph library (BGL)].  The BGL is quite generic and somewhat akward to use, which is why we have wrapped it.  The decision to include a graph library stems from it being useful for at least one other important application: describing a molecule's topology.  Details pertaining to usage of the graph should consult the graph library's documentation.


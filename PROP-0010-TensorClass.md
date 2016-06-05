# PROP 0010 - Tensor Class

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | An object to simplify writing equations   |
| Status:        | In development                            |


## Purpose
Electronic structure theory is naturally expressed in the language of multidimensional arrays, or as physicists and chemists usually call them, tensors.  Hence one of the core classes of any sort of electronic structure theory program will be the tensor class.  A tensor class's main goal is to facilitate the derivations of new theories by allowing one to simply write equations such as:
```
F["i,j"]=H["i,j"]+G["i,j"];
```
The tensor class will provide the foundation for nearly all actual computations so it is important that it is robust, extensible, and well encapsulated.

## Proposed Tensor structure

I propose a tensor "class" that conceptually has this structure:

![Alt text](http://g.gravizo.com/g?
  digraph G {
    aize ="4,4";
    UserAPI [label="User API",shape=box];
    TensorAPI [label="Tensor API",shape=box];
    UserAPI -> TensorAPI;
    node [shape=box,label="SharedAPI"]
    TensorAPI -> SharedAPI;
    node [shape=box,label="SharedImpl"]
    SharedAPI->SharedImpl;
    node [shape=box,label="DistributedAPI"]
    TensorAPI->DistributedAPI;
    node [shape=box,label="DistributedImpl"]
    DistributedAPI->DistributedImpl;
  }
)

Each API is meant to abstract away the details of the API below it.  For the inner APIs it is probably best to adapt a relatively simple API like [BTAS](http://itensor.org/btas/).

* User API is what the user goes through.
  * This is where we implement things we want and are unlikely to be found in all tensor libraries, like views and spatial symmetry.
* Tensor API is the generic mathematical guts of the tensor.
   * This is a "simple" class in terms of functionality.
   * Main purpose is to abstract away the differences between shared v. distributed memory
* Shared API provides an interface that abstracts away which particular shared library implementation we are using
* Distributed API abstracts away which particular distributed library we are using

## Shared and Distributed APIs
It's probably easiest to work backwards in terms of what we want from each API. I expect to use existing
libraries for the actual implementations. The implementations are what actually perform the contractions, permutations, etc. Adding support for a new implementation means writing an adaptor class that forces it into our Shared/Distributed API.  

For shared implementations there is:

* [libtensor](https://github.com/epifanovsky/libtensor)
  * Krylov group
  * Currently powering Q-Chem's coupled cluster codes
  * C++ 
  * Supports spatial and permutational symmetry
  * "Open Source" (actually has Q-Chem dependency, but Anna has told Ben this will go away in a subsequent clean-up)
* [Eigen Tensor](http://eigen.tuxfamily.org/index.php?title=Tensor_support)
  * In case the name didn't give it away, it's from Eigen
  * C++ 
  * Expression templates
  * Permutational symmetry
  * No Einstein notation
  * All tensors are dense
* [Ambit](https://github.com/jturney/ambit)
  * Will let Jet describe

For distributed implementations there is:

* [Cyclops Tensor Framework](https://github.com/solomonik/ctf)
  * Solomonik group
  * Einstein notation (limited to single letter)
  * Expression templates
  * Sparsity
  * Permutational symmetry
* [Tiled Array](https://github.com/ValeevGroup/tiledarray)
  * Valeev group
  * Einstein notation (mulitple letter indices allowed)
  * Expression templates
  * No symmetry support
  * Lazy evaluation pseduo supported (wiki desribes how to implement your own)

## What a Tensor API Needs To Do
 The Tensor API provides a common interface to the underlying Distributed and Shared APIs. It also serves as an insultating layer protecting our added features from the actual features of the libraries.  I think it should do the following:
 
 * Support arbitrary rank
   * Honestly, it doesn't make much sense to only support up to say rank 4 as by that point generalization to higher ranks   is trivial
   * Note many of the existing libraries assume this is template non-type parameter
 * Support arbitrary data
   * Although we usually think about everything as doubles, floats have a role in optimization of routines, finally block    tensors are naturally written as tensors of tensors so support for that simplifies development of those as well.
   * Complex numbers are a touchy subject since in C++ they are [inefficient](https://software.intel.com/en-us/forums/intel-c-compiler/topic/309133), but they certainly have roles in electronic structure theory
   * Again, by time you support all of the above you may as well support arbitrary types.
   * Nearly all existing libraries are templated on the type so we get this for free
 * Einstein notation
   * It dramatically simplifies the interface, is natural, and hides strings of calls to permute etc.
   * One thing to be careful here is that C++ does not allow strings to be non-type template parameters.  This restricts     the amount of compile time optimizations that can occur by deferring them to runtime (reason why Eigen and its tensor   extension do not support Einstein notation at the moment, read about it [here](https://bitbucket.org/eigen/eigen/pull-requests/124/einstein-notation-for-tensor-module/diff)).
   * Limiting them to single letter indices may facilitate future compile time optimizations
   * This may need implemented up a level in the user API.
 * BTAS interface
   * Although it is not clear that any existing tensor library actually follows this, BTAS is really about ensuring
     STL-like interfaces which is important for interacting with standard STL algorithms
   * At least in my head, BTAS will simplify the problem of writing the user-defined interface by allowing access to the STL algorithms
 * Expression templates
   * It is very difficult to implement Einstein-like noation without these
   * We will likely have to implement them as reusing the ones in the implementations will be difficult
     * Really not that hard if you have familiarity with templates
   * Again, may belong up a level.
 * Lazy evaluation
   * The C++-like way to abstract away data access patterns is lazy evaluation.  This allows for reading data from disk,
     reading whole blocks at a time (think shell quartets), or on-the-fly evaluation of elements (think energy denominators)
   * Unfortunately support for this is vague in most libraries so we're probably stuck doing it.
     * Again, it's basic template semantics and not terribly hard as long as you consider it from the start
   * I suspect this needs to happen at this level as it would be this API that needs to determine how to act when given
     an expression for a tensor.

## User API
 This is what 99% of people will see.  
 * Named subspaces, for example ability to grab the "occupied-occupied" block
 * Either implements or does not impede expression templates
 * Does not impede lazy evaluation 
 * Python wrapping happens here
 
## Where do routines go?
One thing to keep in mind is serialization.  Whether we are sending the tensor over a network or writing it disk it needs to be serialized.  The more you add to a class the harder this is.  The BTAS interface really only defines basic operations like addition, contraction, etc.  In addition to that I propose element-wise operations be added (i.e. divide the whole tensor by a number, add 2 to every element, etc.).  Pretty much every other routine like diagonalization or decompositions should be seperate classes or free functions.  This is because they can go through the public interfaces and don't need to be part of the class.  Furthermore they could then be reused with classes other than the tensor class (assuming the other class meets the interface requirements).

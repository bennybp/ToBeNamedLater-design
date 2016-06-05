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

* User API is what the user goes through.
  * This is where we implement things we want and are unlikely to be found in all tensor libraries, like views and spatial symmetry.
* Tensor API is the generic mathematical guts of the tensor.
   * This is a "simple" class in terms of functionality.
   * Main purpose is to abstract away the differences between shared v. distributed memory
* Shared API provides an interface that abstracts away which particular shared library implementation we are using
* Distributed API abstracts away which particular distributed library we are using

## Shared and Distributed APIs
It's probably easiest to work backwards in terms of what we want from each API.  The shared and distributed APIs should have common interfaces, possibly the [BTAS](http://itensor.org/btas/) interface.  I expect to use existing
libraries for the actual implementations.  The implementations are what actually perform the contractions, permutations, etc.  It may be worth 

For shared there is:

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

For distributed there is:

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
 The Tensor API provides a common interface to the underlying Distributed and Shared APIs.  For this section libraries refer to the choices for the distributed and shared APIs.  It also serves as an insultating layer protecting our added
 features from the actual features of the libraries.  I think it should do the following:
 
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
 * 
   * Although it is not clear that any existing tensor library actually follows this, BTAS is really about ensuring
     STL-like interfaces which is important for interacting with standard STL algorithms
   * At least in my head, BTAS will simplify the problem of writing the user-defined interface by allowing 
 * Expression templates
   * It is very difficult to implement Einstein-like noation without these
 * Lazy evaluatoin
   * The C++-like way to abstract away data access patterns is lazy evaluation.  This allows for reading data from disk,
     reading whole blocks at a time (think shell quartets), or on-the-fly evaluation of elements (think energy denominators)
   * Technically doesn't require expression templates, but it is again simplified by them
 * Named subspaces, for example ability to grab the "occupied-occupied" block
 * Expression templates
 * Lazy evaluation (as long as Tiled Array is backend)
 * Wrapper to existing tensor codes: Tiled Array implemented, Cyclops Tensor Framework is a TODO
 * Data locality generally hidden from the developer, although they can specify a preferred storage mechanism.
 * Memory is managed by tensor
 * Paralell filling
 * Compile time error checking

## Interfacial details
As written the tensor can be used as follows:
```C++
Tensor<2> A(10,10);//Declares a 10 by 10 matrix of doubles
Tensor<2,complex<double> > B(10,10);//Declares a 10 by 10 matrix of complex values
Tensor<3> C(10,10,10);//Declares a 10 by 10 by 10 rank 3 tensor of doubles

//Fill A with 0
A.Fill(0.0);

//Fill C in a more complicated fashion
Tensor<3>::iterator Cijk=C.begin(),CEnd=C.end();
for(;Cijk!=CEnd;++Cijk){
  
  //Returns an index like (1,2,3)
  const std::array<size_t,3>& Index=Cijk.Index();
  
  //Sum the index
  size_t sum=0;
  for(size_t i=0;;i<3;++i)sum+=Index[i];
  
  //Set the current value to the sum
  *Cijk=sum;
}

//Contract A and C along first dimension
tensor<3> D(10,10,10);
D["i,j,k]=A["i,l"]*C["l,j,k"];

//Contract A and C along second two dimensions
tensor<1> E(10);
E["i"]=A["k,l"]*C["i,k,l"];

```

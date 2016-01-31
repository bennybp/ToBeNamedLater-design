# PROP 0010 - Tensor Class

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | An object to simplify writing equations   |
| Status:        | In development                            |


## Purpose
Electronic structure theory is naturally expressed in the language of multidimensional arrays, or as physicists and chemists usually call them, tensors.  This class is designed to greatly facilitate the evaluation and parallelization of equations involving tensors, by automatically handeling much of the technical details behind the scenes, given only the equation.  For example, building the Fock matrix is now simply:
```
F["i,j"]=H["i,j"]+G["i,j"];
```
regardless of where the data lives, how it is stored (dense/sparse/replicated/distributed/cyclically/etc.), and the dimensions of the tensors (i.e. this equation works for both RHF and UHF).  At the moment we are intending to use Ryan's tensor wrapper and this document focuses on that.

## Implementation Features
 * Arbitrary rank tensors from 0 to the maximum-unsigned-integer-your-computer-can-handle
 * Arbitrary data (well at the very least double and complex)
 * Arbitrary length indices
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

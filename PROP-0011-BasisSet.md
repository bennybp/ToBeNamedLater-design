# PROP 0011 - Basis Set

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## Purpose

To define standard basis set functionality

## API

The minimum public basis function shell interface will contain functions that do the following:

<!---
![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for a contracted shell
*/
class Shell{
public Shell();
public int L();
public int NPrims();
public int NGeneral();
public void SetExp();
public double Coef();
public double Exp();
public double Comp();
}
)
-->

* **`Shell(int l,int prim,int ngen, double x,double y, double z)`** - Create a shell, with angular momentum `l`, located at cartesian coordinate (x,y,z) (in a.u.), with `nprim` primitives and `ngen` general contractions
* **`double Coord(int i)const`** - returns the i-th Cartesian component in a.u.
* **`int L()const`** - returns the angular momentum, in a.u.
* **`double Coef(int i,int j)const`** - returns the expansion coefficient for the i-th primitive of the j-th general contraction
* **`double Exp(int i)const`** - returns the i-th primitive's exponent in a.u.
* **`bool IsSpherical()const`** - Is this shell spherical or cartesian?
* **`int NPrim()const`** - returns the number of primitives in this shell
* **`int NGeneral()const`** - returns number of general contractions in this shell
* **`int NContractedCartesian()const`** - Returns the number of contracted cartesian functions
* **`int NContractedSpherical() const`** - Returns the number of contracted spherical basis functions
* **`int NContractedBasisFunction() const`** - Returns the number of basis functions (NContractedCartesian() or NContractedSpherical(), depending on IsSpherical())
* **`int NPrimitiveCartesian()const`** - Returns the number of primitive cartesian functions
* **`int NPrimitiveSpherical() const`** - Returns the number of primitive spherical basis functions
* **`int NPrimitiveBasisFunction() const`** - Returns the number of basis functions (NPrimitiveCartesian() or NPrimitiveSpherical(), depending on IsSpherical())
* **`void SetExp(double Exp,int i)`** - sets the i-th primitive's exponent to Exp
* **`void SetCoef(double Coef,int i,int j)`** - sets the i-th primitive of the j-th general contraction to Coef
* **`void SetSpherical(bool isspherical)`** - Sets if this shell is spherical or not

### Points of discussion

* Normalization? I think, in general, they should be *unnormalized*. That way, each code can
normalize how they see fit.
  * Normalization function with callback? `Normalize([](Shell & s){normalize...});` ? 
* Will need some logic for special values of `l` (sp, spd)
  * How to denote these? Negative values of AM?
* Should it be required that data is stored contiguously?
  * Exponents and coefficients within a shell are generally required to be contiguous,
    but not necessarily between all shells. That might be nice, though
* NContractedCartesian would usually return (nprim+1)(nprim+2)/2 * ngeneral, but with some handling of sp, etc
  * Similar for NContractedSpherical, NPrimitiveCartesian, NPrimitiveSpherical
* Iterating over primitives would be kind of awkward with the general contractions
  * Iterate over general contractions, then over primitives?
  * This would be low priority

## Basis Set

The minimum public interface of the BasisSet class should be:

<!---
![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for a BasisSet
*/
class BasisSet{
public Shell GetShell();
public int MaxL();
public int NPrims();
public int NShells();
public void AddShell();
public Molecule Combine();
public Molecule SetDiff();
public Molecule Intersection();
}
)
-->

* **`void AddShell(const Shell& BF)`** - Adds a basis function to the basis set
* **`int NShells()const`** - returns the total number of shells
* **`int NGeneralShells()const`** - returns the number of generally contracted shells
* **`bool HasSpherical()const`** - Are there any shells that are spherical
* **`Shell GetShell(int i)const`** - returns the i-th shell in the basis set as the Shell class
* **`int MaxL()const`** - returns the maximum angular momentum, in a.u.
* **`int NPrims()const`** - returns the total number of primitives
* **`int MaxNPrim()const`** - returns the maximum number of primitives contained in a shell
* **`int NContractedCartesian()const`** - Returns the number of contracted cartesian functions
* **`int NContractedSpherical() const`** - Returns the number of contracted spherical basis functions
* **`int NContractedBasisFunction() const`** - Returns the number of contracted basis functions (depends on IsSpherical() for each shell)
* **`int NPrimitiveCartesian()const`** - Returns the number of primitive cartesian functions
* **`int NPrimitiveSpherical() const`** - Returns the number of primitive spherical basis functions
* **`int NPrimitiveBasisFunction() const`** - Returns the number of basis functions (NPrimitiveCartesian() or NPrimitiveSpherical(), depending on IsSpherical())
* **`int MaxNContractedCartesian()const`** - Returns the maximum number of contracted cartesian functions in a shell
* **`int MaxNContractedSpherical() const`** - Returns the maximum number of contracted spherical basis functions in a shell
* **`int MaxNContractedBasisFunction() const`** - Returns the maximum number of basis functions (depends on IsSpherical() for each shell)
* **`int MaxNPrimitiveCartesian()const`** - Returns the maximum number of primitive cartesian functions in a shell
* **`int MaxNPrimitiveSpherical() const`** - Returns the maximum number of primitive spherical basis functions in a shell
* **`int MaxNPrimitiveBasisFunction() const`** - Returns the maximum number of basis functions (depending on IsSpherical() for each shell)
* **`BasisSet Combine(const BasisSet& Other)const`** - returns the union of this BasisSet and other
* **`BasisSet SetDiff(const BasisSet& Other)const`** - returns the set difference (elements in the first set that are not in the second set) of this and other
* **`BasisSet Intersection(const BasisSet& Other)const`** - returns the shell common to this and other
* **`begin(), end()`** - Iterate over shells
 
### Points of discussion

* Many of the functions listed above are simple convenience functions and generally use other (public) functionality
* Similar discussion as with molecule about whether "set" operations return new objects
* Shells need an index? (similar to Atoms?)
* Need rotate, translate, etc. Would always need to modify this as well as its corresponding Molecule
  * Same with getting fragments, etc
  * And symmetry...
* Future proof with `long` type?

# PROP 0009 - Basis Set

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   |                                           |
| Status:        | In development                            |
 

## API
The philosophy behind this API is similar to the molecule class, so see there for more detail.

The minimum public basis function interface will contain functions that do the following:

![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for a contracted basis function
*/
class BasisFunction{
public BasisFunction();
public int L();
public int NPrims();
public void AddPrim();
public double Coef();
public double Exp();
public double Comp();
}
)

* `BasisFunction(int l,double x,double y, double z)` makes a basis function, with angular momentum l, located at {x,y,z}
* `int L()const` returns the angular momentum, in a.u.
* `int NPrims()const` returns the number of primitives
* `void AddPrim(double& Coef,double& exp)` adds a new primitive with the given coefficient and exponent.  Must be by reference so that general contractions can literally point to the same set of exponents (is this necessary/sufficient?)
* `double Coef(int i)const` returns the i-th primitive's expansion coefficient in a.u.
* `double Exp(int i)const` returns the i-th primitive's exponent in a.u.
* `double Comp(int i)const` returns the i-th Cartesian component in a.u.

The minimum public interface of the BasisSet class should be:

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
public BasisFunction GetBasisFunction();
public int MaxL();
public int NPrims();
public int NBasisFunctions();
public void AddBasisFunction();
public Molecule Combine();
public Molecule SetDiff();
public Molecule Intersection();
}
)

* `BasisFunction GetBasisFunction(int i)const` returns the i-th basis function in the basis set as the BasisFunction class
* `int MaxL()const` returns the maximum angular momentum, in a.u.
* `int NPrims()const` returns the total number of primitives
* `int NBasisFunctions()const` returns the total number of basis functions
* `void AddBasisFunction(const BasisFunction& BF);` Adds a basis function to the basis set
* `BasisSet Combine(const BasisSet& Other)const` returns the union of this BasisSet and other
* `BasisSet SetDiff(const BasisSet& Other)const` returns the set difference (elements in the first set that are not in the second set) of this and other
* `BasisSet Intersection(const BasisSet& Other)const` returns the basis functions common to this and other
 
One is free to provide additional functionality beyond this, but not required.
 

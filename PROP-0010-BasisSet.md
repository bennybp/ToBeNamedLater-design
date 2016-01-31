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

* `Shell(int l,double x,double y, double z)` makes a shell, with angular momentum l, located at {x,y,z}
* `int L()const` returns the angular momentum, in a.u.
* `int NPrims()const` returns the number of primitives
* `int NGeneral()const` returns 1 for segmented basis functions and NPrims (?) for general basis sets
* `void SetExp(double Exp,int i)` sets the i-th primitive's exponent to Exp
* `void SetCoef(dobule Coef,int i,int j)` sets the i-th primitive of the j-th basis function to Coef
* `double Coef(int i,int j)const` returns the expansion coefficient for the i-th primitive of the j-th basis function  in a.u.
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

* `Shell GetShell(int i)const` returns the i-th shell in the basis set as the Shell class
* `int MaxL()const` returns the maximum angular momentum, in a.u.
* `int NPrims()const` returns the total number of primitives
* `int NShells()const` returns the total number of shells
* `void AddShell(const Shell& BF);` Adds a basis function to the basis set
* `BasisSet Combine(const BasisSet& Other)const` returns the union of this BasisSet and other
* `BasisSet SetDiff(const BasisSet& Other)const` returns the set difference (elements in the first set that are not in the second set) of this and other
* `BasisSet Intersection(const BasisSet& Other)const` returns the shell common to this and other
 
One is free to provide additional functionality beyond this, but not required.
 

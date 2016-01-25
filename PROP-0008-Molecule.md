# PROP 0008 - Molecule

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Our chemical system of interest           |
| Status:        | In development                            |
 

## Purpose
 In chemistry we are interested in molecules and their environments and want to perform compuations on these "systems".  The system may contain multipole moments, implicit solvation (usually through multipole moments), electric fields, magnetic fields, extra basis functions, points of reference, etc.  Because the name "system" is confusing in a computer science context we will continue to call the main class molecule even though it is more than just atoms.
 
## APIs
The molecule will ultimately be made up of a set of atoms.  Here we layout what the minimum functionality should be for these two classes for others wishing to be consistent with our API.  When implementing them I encourage developers to follow language precident so long as all of these operations exist.  For example, the `double Comp(int i)` function of the Atom class, described shortly, is in C++ most naturally expressed as `double operator[](int i)`.  The point is as long as I can get the i-th component of your atom I don't care what you call the function.  In the next section we will describe how the atom and molecule classes are handled within the current module project.  Aside from the following API both atom and molecule must have a mechanism for deep copy.

Anyways, the minimum public atom interface will contain functions that do the following:

![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for an atom
*/
class Atom{
public Atom();
public double Mass();
public double Charge();
public int Mult();
public int Z();
public double NElectrons();
public double Comp();
}
)

* `Atom(int Z,double x,double y, double z)` sets the atom up with atomic number Z located at {x,y,z}
* `double Mass()` returns the mass, in a.u.
* `double Charge()` returns the charge of the atom, in a.u. In general this should be Z less the number of electrons, and should be a double to support say MM charges
* `int Mult()` returns the multiplicity of the atom
* `int Z()` returns the atomic number
* `double NElectrons()` returns the number of electrons, must be a double to allow for fractional numbers of electrons
* `double Comp(int i)` returns the i-th Cartesian component in a.u.

The minimum public interface of the molecule class should be:

![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for a molecule
*/
class Molecule{
public Atom GetAtom();
public double Charge();
public int Mult();
public double NElectrons();
public double Comp();
public Molecule Combine();
public Molecule SetDiff();
public Molecule Intersection();
}
)

* `Atom GetAtom(int i)` returns the i-th atom in the molecule as the Atom class
* `double Charge()` returns the charge of the molecule, in a.u. and as double to support fractional charges
* `int Mult()` returns the multiplicity of the molecule
* `double NElectrons()` returns the number of electrons, must be a double to support fractional numbers
* `double Comp(int i,int j)` returns the j-th Cartesian component of the i-th atom , in a.u.
* `Molecule Combine(Molecule Other)` returns the union of this molecule and other
* `Molecule SetDiff(Molecule Other)` returns the set difference (elements in the first set that are not in the second set) of this and other
* `Molecule Intersection(Molecule Other)` returns the atoms common to this and other
 
One is free to provide additional functionality beyond this, but not required.

## Module Project Interface
Ultimately the goal of this interface is to have a syntax like:
```C++
//Declare a CO2 molecule along the z-axis
Atom C(6,{0.0,0.0,0.0});
Atom O1(8,{0.0,0.0,-2.19});
Atom O2(8,{0.0,0.0,2,19});
Molecule CO2({C,O1,O2});

/**** Sometime later, in some other function***/

//Grab the C=O1 bond
Molecule CO({CO2[0],CO2[1]});

//Grab the O2 atom
Molecule O2({CO2[2]});

//Make another CO2
```

## Features
 * Recursive fragments
 * Iteration over all atoms within a fragment, including the top-level "Molecule"
 * Iteration over all sub-fragments within a fragment (right now this requires a MathSet<MathSet<Atoms> >)

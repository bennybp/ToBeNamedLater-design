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
public void SetMass();
public double Charge();
public void SetChargeAndMult();
public int Mult();
public string Symbol();
public int Z();
public double NElectrons();
public void SetNElectrons();
public double Comp();
}
)
General notes atomic number and coordinates are expected to remain constant from construction so no setters are defined.
Otherwise the functions are:
* `Atom(int Z,double x,double y, double z)` sets the atom up with atomic number Z located at {x,y,z}
* `double Mass()const` returns the mass, in a.u.
* `void SetMass(double m)` sets the mass to m (m in a.u.)
* `double Charge()const` returns the charge of the atom, in a.u. In general this should be Z less the number of electrons, and should be a double to support say MM charges
* `void SetChargeAndMult(double q,int m)` sets the charge to q (q in a.u.) and multiplicity to m
* `int Mult()const` returns the multiplicity of the atom
* `string Symbol()const` returns the atomic symbol of the atom
* `int Z()const` returns the atomic number
* `void SetNElectrons(double N)` sets the number of electrons to N
* `double NElectrons()const` returns the number of electrons, must be a double to allow for fractional numbers of electrons
* `double Comp(int i)const` returns the i-th Cartesian component in a.u.

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
public void AddAtom();
public double Charge();
public int Mult();
public double NElectrons();
public Molecule Combine();
public Molecule SetDiff();
public Molecule Intersection();
}
)

* `Atom GetAtom(int i)const` returns the i-th atom in the molecule as the Atom class
* `void AddAtom(const Atom& i)` adds atom i to the molecule
* `double Charge()const` returns the charge of the molecule, in a.u. and as double to support fractional charges
* `int Mult()const` returns the multiplicity of the molecule
* `double NElectrons()const` returns the number of electrons, must be a double to support fractional numbers
* `double Comp(int i,int j)const` returns the j-th Cartesian component of the i-th atom , in a.u.
* `Molecule Combine(const Molecule& Other)const` returns the union of this molecule and other
* `Molecule SetDiff(const Molecule& Other)const` returns the set difference (elements in the first set that are not in the second set) of this and other
* `Molecule Intersection(const Molecule& Other)const` returns the atoms common to this and other
 
One is free to provide additional functionality beyond this, but not required.  For languages not supporting `const` the spirit of the const is expected to be upheld, i.e. if an argument is a `const &` you better not change it.

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

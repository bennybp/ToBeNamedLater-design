# PROP 0009 - Molecule

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Our chemical system of interest           |
| Status:        | In development                            |
 

## Purpose

 In chemistry we are interested in molecules and their environments
 and want to perform compuations on these "systems".  The system may
 contain multipole moments, implicit solvation (usually through multipole
 moments), electric fields, magnetic fields, extra basis functions,
 points of reference, etc.  This proposal really just focuses on the
 molecule aspect of the system.  The remainder, which we term Space will
 be the subject of another proposal.
 
## APIs

The molecule will ultimately be made up of a set of atoms (or atomic centers).
Here we layout what the minimum interface for a molecule should be, for the `Atom` and `Molecule` classes.
Developers wishing to adopt their code to this interface should try to be consistent with our API.
When implementing them I encourage developers to follow language precedent so long as all of these operations exist.
For example, the `double Comp(int i)` function of the Atom class, described shortly, is in C++ most naturally expressed as `double operator[](int i)`.

<!---
The point is as long as I can get the i-th component of your atom I don't care what you call the function.  Yes, a true API specifies signatures, but I think a more realistic first step for our field is just to ensure we have common functionality.  In the next section we will describe how the atom and molecule classes are handled within the current module project.  Aside from the following API both atom and molecule must have a mechanism for deep copy, shallow copy is optional.
-->

### Atom
The minimum public atom interface will contain functions that do the following:

<!---
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
-->

General notes atomic number and coordinates are expected to remain constant from construction so no setters are defined (*debatable*).
When an atomic number is provided, reasonable defaults for all properties are expected to be set (such as the most common isotope).

Otherwise the functions are:
* **`Atom(int Z,double x,double y, double z)`** - Constructor. Creates an atom with atomic number Z located at cartesian coordinates (`x`,`y`,`z`) (in bohr, au)
* **`int Z()const`** - Returns the atomic Z number
* **`int Isotope()const`** - Returns the isotope number (number of protons + neutrons?)
* **`double Coord(int i)const`** - Returns the i-th Cartesian component in a.u.
* **`std::string Symbol()const`** - Returns the atomic symbol of the atom
* **`std::string Name()const`** - Returns the elemental name of the atom
* **`double Mass()const`** - Returns the atomic mass, in a.u.
* **`double IsotopeMass()const`** - Return the mass of the currently-selected isotope
* **`double Charge()const`** - Returns the charge of the atom, in a.u. In general this should be Z less the number of electrons, and should be a double to support say MM charges
* **`int Mult()const`** - Returns the multiplicity of the atom
* **`double NElectrons()const`** - Returns the number of electrons. Double to allow for fractional occupation

* **`void SetIsotope(int i)`** - Set the isotope number
* **`void SetCoords(double x, double y, double z)const`** - Set the coordinates of the atom
* **`void SetMass(double m)`** - Change the atomic mass to `m` (m in a.u.)
* **`void SetChargeAndMult(double q,int m)`** - Change the charge to `q` (in a.u.) and multiplicity to `m`
* **`void SetNElectrons(double N)`** - Sets the number of electrons to `N`. Double to allow for fractional occupation

#### Points of discussion

* Allow changing of `Z` or of the coordinates
* Enforcing units to be a.u. (bohr) for coordinates.
* Should the class have a default constructor
* Should the copy constructor and copy assignment be a deep copy
* Name of Coord(s) functions, and should `operator[]` be overloaded
  to mean the same thing
* Should have indexing element?

### Molecule

The minimum public interface of the molecule class should be:

<!---
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
public void SelfCombine();
public void SelfSetDiff();
public void SelfIntersection();
}
)
-->

Again, I expect the molecule to provide reasonable defaults to charge
and multiplicity, which are updated as atoms are added (removing atoms
is optional, but if supported then this should also be accounted for).

* **`Atom GetAtom(int i)const`** - Returns the `i`-th atom in the molecule as the Atom class
* **`double Charge()const`** - Returns the charge of the molecule, in a.u. (double to support fractional charges)
* **`double NElectrons()const`** - Returns the number of electrons. Must be a double to support fractional numbers
* **`int Mult()const`** - Returns the multiplicity of the molecule


* **`void AddAtom(const Atom& i)`** - Adds atom i to the molecule

* **`double Coord(int i,int j)const`** returns the `j`-th Cartesian component of the `i`-th atom , in a.u.

* **`void SelfSetAdd(const Molecule& Other)`** - Makes current molecule the union of this molecule and other
* **`void SelfSetDiff(const Molecule& Other)`** - Makes current molecule the set difference (the elements unique to the first set) of this and other
* **`void SelfIntersection(const Molecule& Other)`** - Removes from this molecule any atoms not in other
* **`void Rotate(const LocalMatrix & mat)`** - Rotate the molecule given a rotation matrix
* **`void Translate(const LocalVector & v)`** - Translate the molecule
* **`begin(), end()`** - Iterate over all atoms
 
One is free to provide additional functionality beyond this, but not
required.  For languages not supporting `const` the spirit of the const
is expected to be upheld, i.e. if an argument is a `const &` you better
not change it.  Similarly if something is returning

**Note** - As written, there should be no assumption (at the interface level) as to the layout of
the data (contiguousness, etc) to the developer using the library. I.e., there should be no `&GetAtom[0]` type stuff.

#### Points of discussion

* Should we allow changing of atom data via `GetAtom` and then changing it? Or should `GetAtom` return a copy
  * Ben - Changing my view slightly, and maybe it should be possible. It is generally what is expected
and should be OK as long as parts of the molecule aren't shared with other molecules explicitly.
* Should GetAtom return a reference or a copy? (somewhat depends on above)
  * Copying is relatively cheap and developers should not be doing expensive stuff through Atoms
* Estimating the multiplicity of a molecule via multiplicity of atoms is not trivial.
* How is symmetry stored
* No `SetCharge()`?
* Naming of set operations is awkward
* Should we allow large-scale changing to `this`? (ie, should `SelfSetAdd` or Rotate, Translate, etc, functions return a new Molecule rather than changing this).
  * `ReplaceAtom` ?
* If no assumptions can be made about the contiguousness of the data, then the Rotate and Translate functions
would probably have to be part of the class (to take advantage of contiguous data if it does exist).
  * Or could have a `RotateMolecule` class that is friends with Molecule? If that is so tightly coupled to the Molecule, then it should be part of molecule
  * Or `RotateMolecule` can be an inner class
* Fragments? Iterating over fragments?



## Features (of our implementation)
Ultimately the goal of our interface is to have a syntax like:

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
Molecule CO2_2=CO+O2;

//Print the carbon atom out
std::cout<<CO[0]<<std::endl;

```

 * Arbitrary nested fragments, that is each fragment is a perfectly good molecule and can be fragmented as well
 * Effeciency: memory for each atom is contigious allowing for BLAS calls on coordiantes, masses, charges, etc. This is done without duplicating data (basically there's some messy pointer redirection the user need not concern themselves with).

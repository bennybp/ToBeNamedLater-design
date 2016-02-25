# PROP 0009 - Molecule (and Atom)

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Our chemical system of interest           |
| Status:        | In development                            |

## Content
 1. [Purpose](#purpose)
 2. [APIs](#apis)
    - [Atom](#atom)
    - [Atom Discussion](#atom-discussion)
    - [Molecule](#molecule)
    - [Molecule Discussion](#molecule-discussion)
 3. [Features of Our Implementation](#features-of-our-implementation)
    - [Our Atom Class](#our-atom-class)
    - [Our Molecule Class](#our-molecule-class)

## Purpose

 In chemistry we are interested in molecules and their environments
 and want to perform compuations on these "systems".  The system may
 contain multipole moments, implicit solvation (usually through multipole
 moments), electric fields, magnetic fields, extra basis functions,
 points of reference, etc.  This proposal really just focuses on the
 molecule aspect of the system.  The remainder, which we term Space will
 be the subject of another proposal.
 
#### APIs

The molecule will ultimately be made up of a set of atoms (or atomic centers).
Here we layout what the minimum interface for a molecule should be, for the `Atom` and `Molecule` classes.
Developers wishing to adopt their code to this interface should try to be consistent with our API.
When implementing your classes I encourage you to follow language precedent so long as all of these operations exist.
For example, the `double Comp(int i)` function of the Atom class, described shortly, is in C++ most naturally expressed as `double operator[](int i)`.

Ryan - I think there are two issues at play, what should our Atom/Molecule class do? And what functionality do we expect from other people's atom/molecule classes.  This section focuses on the latter.  Thus I motion we move some the following discussions to the next section.

<!---
The point is as long as I can get the i-th component of your atom I don't care what you call the function.  Yes, a true API specifies signatures, but I think a more realistic first step for our field is just to ensure we have common functionality.  In the next section we will describe how the atom and molecule classes are handled within the current module project.  Aside from the following API both atom and molecule must have a mechanism for deep copy, shallow copy is optional.
-->

##### Atom
The minimum public atom interface will contain functions that do the following:

(put UML diagram back when design is more solidified)

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
<!--
General notes atomic number and coordinates are expected to remain constant from construction so no setters are defined (*debatable*).
-->

When an atomic number is provided, reasonable defaults for all properties are expected to be set (such as the most common isotope).

Otherwise the functions are:
* **`Atom()`** -Default constructor, atom should be in a reasonable starting state
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
* **`void SetZ(int i)`** - Sets the atomic number

##### Atom Discussion

* Allow changing of `Z` or of the coordinates
  * Ryan - I'm going back on my original statement and think they should be modifiable.  We should rely on const-ness to enforce can/can't modify something (above statements have been modified)
* Enforcing units to be a.u. (bohr) for coordinates.
  * Ryan - All units, everywhere should be a.u. 
* Should the class have a default constructor
  * Ryan - I'm a big fan of default constructors 
* Should the copy constructor and copy assignment be a deep copy
  * Ryan - I think they should be sha 
* Name of Coord(s) functions, and should `operator[]` be overloaded
  to mean the same thing
  * Ryan -  For our implementation I think `operator[]` should definately be used
* Should have indexing element?
  * Ryan - I really hate the idea of referring to "Atom 1", but also realize it is very convienient sometimes. 

#### Molecule

The minimum public interface of the molecule class should be:

(put UML diagram back when design is more solidified)

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

##### Molecule Discussion

* Should we allow changing of atom data via `GetAtom` and then changing it? Or should `GetAtom` return a copy
  * Ben - Changing my view slightly, and maybe it should be possible. It is generally what is expected
and should be OK as long as parts of the molecule aren't shared with other molecules explicitly.
  * Ryan - I think you should be able to edit the atoms if you have a non-const molecule
* Should GetAtom return a reference or a copy? (somewhat depends on above)
  * Copying is relatively cheap and developers should not be doing expensive stuff through Atoms
  * Ryan - I think it should be a reference with the appropriate const-ness
* No `SetCharge()`?
  * Ryan - Need  this function, was an oversight.
* Naming of set operations is awkward
  * Ryan - They are, but in any reasonable language they would just be operators 
* Should we allow large-scale changing to `this`? (ie, should `SelfSetAdd` or Rotate, Translate, etc, functions return a new Molecule rather than changing this).
  * `ReplaceAtom` ?
  * Ryan - `SetlfSetAdd` is the equivalent to `operator+=` which conventionally returns `*this`.  I envision it more for building up the molecule, rather than working with the existing molecule
* If no assumptions can be made about the contiguousness of the data, then the Rotate and Translate functions
would probably have to be part of the class (to take advantage of contiguous data if it does exist).
  * Or could have a `RotateMolecule` class that is friends with Molecule? If that is so tightly coupled to the Molecule, then it should be part of molecule
  * Or `RotateMolecule` can be an inner class
  * Based on Ben's tests of the time to rotate 50,000,000 atoms, using non-contigious memory (final time <10 seconds) I vote that Rotate/Translate be free functions or something similar as to maintain the sort of data/algorithm seperation
* Fragments? Iterating over fragments?
  * Ryan - For the API I think it's overkill, for our implementation discussed below 
   
<!-- Moved to our section
 Estimating the multiplicity of a molecule via multiplicity of atoms is not trivial.
 How is symmetry stored
-->

## Features of our implementation
We need to flush out some more use cases and also address element access.  This example relies on the "give me Atom 1" mindset.

Ultimately the goal of our interface is to have a syntax like:

```C++
//Within your module you are given a const Molecule
const Molecule YourMolecule;
//which contains the atoms as a MathSet

//This won't work because YourMolecule is const
YourMolecule<<MyNewAtom;

//If you want to add an atom you need to copy YourMolecule
Molecule MyClone(YourMolecule);

MyClone<<MyNewAtom;
/** now only works if MyNewAtom is inside the universe MathSet is associated with
*  
*   More concretely, assume that the molecule in the input file is a water dimer, with the two
*   waters being A and B respectively.  Now pretend YourMolecule is actually A.  If MyNewAtom is
*   any of the atoms in B, then this works.  Usually when one is mainpulating molecules
*   this is what they want to do, include other parts of the input molecule in the
*   current computation or take some of them out.  These sorts of operatations work
*   fine with a non-cost (the copy) molecule.
*
*   Now what happens if you actually want to add the element?  First we make a new universe:
*/
AtomSetUniverse NewUniverse;

//Then we add the atom
NewUniverse<<MyNewAtom;

//Then we add whatever other atoms we want to it as well
NewUniverse<<SomeOtherAtom;

//Finally we make our new molecule
Molecule NewMolecule(NewUniverse);

//This workflow keeps the actual atoms constant within the molecule, but allows fragmentation to occur
//For common molecular operations, such as rotation/translation free-functions exist to do this for you
Molecule MyRotatedMolecule=RotateMolecule(YourMolecule,RotationMatrix);

/*For most modules manipulating the molecule should not be an issue, for example Hartree-Fock should not
 * be manipulating the molecule, but for say a CP correction module we will have to modify it.
 * So how do we do that?  Well again, we need to add elements to a universe (and not a molecule,
 * which is a MathSet)
 */

Atom NewGhost=MakeGhost(MyNewAtom);
//Make ghost is a function that turns MyNewAtom into a ghost atom and then returns it (ensures that
//the internal flags are set consistently.  At the moment ghosts have Z=0, mass=0, no electrons)

//Similarly we have free functions that check if an atom is ghost (again keeps track of the internal flags)
bool GhostQuestionMark=IsGhost(NewGhost);

/* Note that for a system that has ghost atoms, each atom will appear twice in NewUniverse, once as
 * a real and once as a ghost (three times for point charges).  Admittidly this is somewhat odd, but
 * the user shouldn't concern themselves with it because it ensures you get the syntax you expect:
 */

MyClone<<MakeGhost(OxygenFromWaterB);
MyClone<<MakeGhost(Hydrogen1FromWaterB);
MyClone<<MakeGhost(Hydrogen2FromWaterB);
//Where we returned to our water dimer example

```
### Our Atom Class
   * Contains its basis set
   * Contains a nucleus subclass tracking isotope data as well as effective core potentials, finite nucleus, etc.
   * Knows empirical physical data for that element, such as covalent/VDW raddii etc. (implemented via pointer to some const class)
    
### Our Molecule Class
   * Rely on MathSet for nearly all functionality via pimpl idiom
   * Fragmentation through MathSet's `Partition()` function with lambda/functor
   * This is fine for generating them, but now where do they live?  
      * Merit in having a molecule keep track of its fragments.  
      * At the same time, partitioning a set is a layer above the set and thus suggests that the set itself should be agnostic       * Can imagine our fragmented system is actually a MathSet of molecules
   * Symmetry class containing point-group info, symmetry operatrions, character-table etc. (move to Symmetry page?)
      * Thinking towards space groups and generating them from translations and point-groups, on the fly
   * Connectivity data or maybe just a list of bonds, angles, torsions
      * My thought here is that the connectivity data is very useful for say fragmenting 
   * Some mechanism for generating internal coordinates/ Z-matrices, and accessing them.  
      * Could be exterior factory or derived class.
   * Automatic estimation of multiplicity.  
      * Admittidly somewhat non-trivial, but given how much I hate term symbols, it would be great to have it automated.
  
<!--
Arbitrary nested fragments, that is each fragment is a perfectly good molecule and can be fragmented as well
 Effeciency: memory for each atom is contigious allowing for BLAS calls on coordiantes, masses, charges, etc. This is done without duplicating data (basically there's some messy pointer redirection the user need not concern themselves with).
-->

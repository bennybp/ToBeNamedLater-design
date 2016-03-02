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
 



<!---
#### APIs
The molecule will ultimately be made up of a set of atoms (or atomic centers).
Here we layout what the minimum interface for a molecule should be, for the `Atom` and `Molecule` classes.
Developers wishing to adopt their code to this interface should try to be consistent with our API.
When implementing your classes I encourage you to follow language precedent so long as all of these operations exist.

For example, the `double Comp(int i)` function of the Atom class, described shortly, is in C++ most naturally expressed as `double operator[](int i)`.

Ryan - I think there are two issues at play, what should our Atom/Molecule class do? And what functionality do we expect from other people's atom/molecule classes.  This section focuses on the latter.  Thus I motion we move some the following discussions to the next section.

The point is as long as I can get the i-th component of your atom I don't care what you call the function.  Yes, a true API specifies signatures, but I think a more realistic first step for our field is just to ensure we have common functionality.  In the next section we will describe how the atom and molecule classes are handled within the current module project.  Aside from the following API both atom and molecule must have a mechanism for deep copy, shallow copy is optional.
-->


## Atom

The minimum public atom interface will contain functions that do the following:

(put UML diagram back when design is more solidified)


### Constructors

* **`Atom()`** -Default constructor, atom should be in a reasonable starting state
* **`Atom(int idx, Coords xyz, int Z, int isonum, double charge, double multiplicity, double nelectrons)`** - Constructor. Creates an atom with all the information filled in
* **`Atom(int idx, double x, double y, double z, int Z, int isonum, double charge, double multiplicity, double nelectrons)`** - Constructor. Creates an atom with all the information filled in

### Basic properties

Most properties are accessed by conventional getter/setter methods. Getter methods return copies of the
property.
A `CoordType` is a typedef for `std::array<double, 3>`.


| Getter                              |Setter                                 | Description                                                                  |
|-------------------------------------|---------------------------------------|------------------------------------------------------------------------------|
| **`size_t GetIdx() const`**          | (none)                                | Unique atom index (usually input ordering)                                   |
| **`int GetZ() const`**               | **`void SetZ(int)`**                   | Atomic Z number (number of protons)                                          |
| **`int GetIsonum() const`**          | **`void SetIsonum(int)`**              | Isotope number (protons + neutrons)                                          |
| **`double GetMass() const`**         | **`void SetMass(double)`**             | Atomic mass (in amu, weighted average of isotope masses                      |
| **`double GetIsotopeMass() const`**  | **`void SetIsotopeMass(double)`**      | Mass of this particular isotope (in amu)                                     |
| **`double GetCharge() const`**       | **`void SetCharge(double)`**           | Charge on this atom / center                                                 |
| **`double GetMultiplicity() const`** | **`void SetMultiplicity(double)`**     | Electronic spin multiplicity (2S+1)                                          |
| **`double GetNElectrons() const`**   | **`void SetNElectrons(double)`**       | Number of electrons on this atom / center                                    |
| **`CoordType GetCoords() const`**    | **`void SetCoords(const CoordType &)`**<br />**`void SetCoords(double, double, double)`** | Coordinates of this center (in au)    |
| **`double GetCoord(int)`**           | **`void SetCoord(int, double)`**       | Get/Set and individual component of the coordinates                          |

Copy construction and assigment operations (with both references and rvalue references) are required.
Overloads for `operator[]` may be provided for the coordinates, returning a reference.
The comparison operators `operator==` and `operator!=` should also be provided.


### Free functions

The `CreateAtom` free functions are used to create an atom based only on Z and coordinates (and optionally isotope number), which
creates an atom with the other data fill in with reference values.


### Basis function information

Basis functions on the center are to be stored with the other atom information. This is a map of
an arbitrary label string (representing the purpose of the basis set, ie "primary" or "density fitting") to a
vector of shell information.

* **`ShellInfoMap GetAllShells() const`** - Return all basis function information stored with this atom
* **`ShellInfoVector GetShells(std::string) const`** - Get all the shells associated with a particular basis set label
* **`void SetShells(std::string, ShellInfoVector)`** - Set/replace all the shells for a given label
* **`void AddShell(std::string, ShellInfo)`**    - Add a shell to a particular label



## Molecule

A Molecule is a read-only view of a collection of Atoms. This collection of Atoms is called the universe of atoms.

The Molecule is, strictly, **immutable**. Once an atom is added to a universe, it cannot be changed directly.
Once a Molecule is constructed from a given universe, then that universe is immutable when accessed via the
Molecule.  This allows for shallow-copy semantics without needing to worry about other parts of the code
changing the underlying data. Any changes to the Atoms within the Molecule require constructing a new
Molecule.

The only changes allowed for a (non-const) Molecule is the changing of what Atoms (from the universe)
constitute the Molecule.


* **`int NAtoms() const`** - Number of atoms in this Molecule (not the universe)
* **`double GetCharge()const`** - Returns the charge of the molecule, in a.u. (double to support fractional charges)
* **`void SetCharge(double)`** - Set the charge of the molecule, in a.u. (double to support fractional charges)
* **`double GetNElectrons()const`** - Returns the number of electrons. Must be a double to support fractional numbers
* **`void SetNElectrons(double)`** - Set the number of electrons. Must be a double to support fractional numbers


* **`bool HasAtom(size_t) const`** - See if this Molecule contains an Atom with a specific global index
* **`Atom GetAtom(int i)const`** - Returns the `i`-th Atom in the Molecule (by copy).
* **`begin(), end()`** - Iterate over all atoms

* **`Point CenterOfMass(void) const`** - Calculate the mass-weighted
* **`Point CenterOfNuclearCharge(void) const`** - Calculate the Z-weighted center
* **`Molecule Translate(Vector) const`** - Translate the molecule
* **`Molecule Rotate(Matrix) const`** - Rotate the molecule

* **`void Insert(const Atom &) const`** - Inserts an atom (must already be a part of the universe of this Molecule
* **`Molecule Partition(SelectorFunc) const`** - Return a subset of the Molecule based on a selection function. Universe is preserved.
* **`Molecule Transform(TransformerFunc) const`** - Return a subset of the Molecule based on a selection function. Universe is preserved.
* **`Molecule Complement(void) const`** - Returns a Molecule containing all Atoms in the universe that are not part of this Molecule
* **`Molecule Intersection(const Molecule &) const`** - Returns a Molecule containing all Atoms that are both in this Molecule and another Molecule. Must share universes.
* **`Molecule Union(const Molecule &) const`** - Returns a Molecule that combines this Molecule and another Molecule. Must share universes.
* **`Molecule Difference(const Molecule &) const`** - Returns a Molecule that contains the difference between this Molecule and another

* **`BasisSet GetBasisSet(const std::string &) const`** - Get the basis set (with the specified label) associated with this Molecule.


A `SelectorFunc` is a function or functor (or lambda) that takes a `const Atom &` as an argument
and returns true if that atom should be included in the new molecule. Similarly, a `TransformerFunc`
takes a `const Atom &` and returns a new, transformed Atom to be included in the new Molecule.

The copy constructor and assignment operator all shallow-copy the universe to the new Molecule.
 
**Note** - As written, there should be no assumption (at the interface level) as to the layout of
the data (contiguousness, etc) to the developer using the library. I.e., there should be no `&GetAtom[0]` type stuff.


## Fragmentation

Fragmentation information is not included within the Molecule class. Discussions are
ongoing on exactly how a Molecule may be fragmented, but seems fairly settled on
a free function/class/module that returns a map of Molecules based on some criteria (user-defined).
Similar for connectivity.



## Features of our implementation
We need to flush out some more use cases and also address element access.  This example relies on the "give me Atom 1" mindset.

Ultimately the goal of our interface is to have a syntax like:

```C++
/* For concreteness throughout this example we assume the user has given us the water
 * dimer and we label the two waters as A and B.  A's atoms are:
 *
 * OA
 * HA1
 * HA2
 * 
 * and B's are:
 *
 * OB
 * HB1
 * HB2
 */
 
//Within your module you will be given a const Molecule (probably by reference)
//We assume YourMolecule={WaterA,WaterB}
const Molecule YourMolecule;

/* This molecule contains the atoms as a MathSet.  A MathSet aliases the actual
 * elements in it to a more managable type (specifically an integer) and performs
 * all set-like operations on the integers rather than the elements themselves.
 * Because there is a one-to-one mapping between the integers and the elements
 * it's as if we manipulated the elements themself; however, manipulating the
 * integers is far more effecient in terms of speed and memory, which is important
 * for scenarios in which we have lots of molecules floating around (particularly
 * things like the many-body expansion and finite-difference computations).  How
 * the atoms are actually stored is somewhat of an implementation detail.  What's
 * more important is how to use the Molecule class.
 *
 * The molecule class enforces that the actual atoms themselves are const.
 * (We are also assuming for simplicity that the atom indices map to the
 * elements of A than B in the order above):
 */
const Atom& OA=YourMolecule[0];

//You are free to copy the atom
Atom OA_Clone(OA);

//and now you can change it for example we can give it a silly mass:
OA_Clone.SetMass(255);

//For a number of reasons, the most important of which is that YourMolecule
//is const, this won't work:
YourMolecule<<OA_Clone; //<---Won't work, won't even compile

//So let's pretend you are set on adding this atom so you try copying:
Molecule MyClone(YourMolecule);

//This Still won't work:
MyClone<<OA_Clone;//Will throw element out of range

/* To understand why this won't work you have to recall that we have a MathSet
 * and not a Universe.  We can only add new elements to a universe.  We can
 * only add elements in the universe to the MathSet.  Our universe is:
 *
 * {OA,HA1,HA2,OB,HB1,HB2}
 *
 * which clearly doesn't contain OA_Clone.  Note we could do (although it
 * won't actually do anything since OA is already in our molecule):
 */
 MyClone<<YourMolecule[0];
 
 /* At this point manipulations of the Molecule may seem limited, but as
  * far as I can tell the most common manipulations are of the form:
  *
  * (Discussion note, Molecule needs to be default constructable for this
  * to work)
  */
   Molecule WaterA,WaterB;
   WaterA=NewMolecule.Fragment(
      [](const Atom& AnAtom){return AnAtom.Index()<3;});
   WaterB=NewMolecule.Fragment(
      [](const Atom& AnAtom){ return AnAtom.Index()>2;});
  
  /* These operations do exactly what you expect, they split the dimer
   * into it's two monomers.  In the lambda we could have used any of the
   * Atom's properties to perform the spilt, e.g. split by oxygen vs. hydrogen,
   * or by distance from some point.
   *
   * Hopefully you are beginning to see that this syntax is actually quite
   * powerful, but we still haven't addressed how we actually change an atom.
   * Rather than trying to add our silly clone.  Let's set up the counterpoise
   * correction.  Recall that we can only add new atoms to a universe, so we
   * make a new universe.
   */
  AtomSetUniverse NewUniverse;

  //Now we add the original atoms to it (these atoms will be deep copied)
  for(const Atom& AtomI : YourMolecule){
     NewUniverse<<AtomI;
     //Now the key step, we add ghost versions as well
     NewUniverse<<MakeGhost(AtomI);
  }
  /* The MakeGhost function is a predefined free-function that sets
   * the appropriate flags for you so that the atom will be recognized
   * as a ghost atom (currently sets Z=0, mass=0, and number of electrons
   * to 0).  You should not attempt to set the flags yourself incase they
   * change, in which case we only need to update the MakeGhost function
   * to propagate the changes.
   */
  
  //And we now turn our universe into a Molecule
  Molecule NewMolecule(NewUniverse);
  
  /* At this point our new universe (and our NewMolcule) look 
   * like (@ denotes a ghost version):
   *
   * {OA,@OA,HA1,@HA1,HA2,@HA2,OB,@OB,HB1,@HB1,HB2,@HB2}
   *
   * I realize this is very non-standard, but the average user will never
   * need to consider this detail and mathematically it actually makes a
   * lot of sense since OA!=@OA, that is they really are different elements of
   * a universe.  
   * 
   * (Discussion note: when are two atoms equal, i.e. can we use the atoms that
   * are currently in WaterA and WaterB or do we have to remake those monomers?)
   *
   * For the puroposes of this demo, we remake the real molecules until the above
   * discussion point is addressed (note that the ghost atoms have odd indices).
   *
   */
   Molecule RealA,RealB;
   RealA=NewMolecule.Fragment(
      [](const Atom& AnAtom){return AnAtom.Index()<5 && AnAtom.Index()%2==0;});
   RealB=NewMolecule.Fragment(
      [](const Atom& AnAtom){ return AnAtom.Index()>5 && AnAtom.Index()%2==0;});

   //Now we add B's atoms as ghosts to A
   WaterA=RealA;
   for(const Atom& AnAtom : RealB)
      WaterA<<MakeGhost(AnAtom);
   
   //and vice versa
   WaterB=RealB;
   for(const Atom& AnAtom: RealA)
      WaterB<<MakeGhost(AnAtom);
   
   /* Note a few things:
    *
    * WaterA=RealA actually changes the universe WaterA is linked to. It does not change
    * the elements within that universe.  Furthermore, since RealA contains
    * a const Universe (through MathSet) this does not allow WaterA to actually change atoms
    * in its new universe.
    *
    * Adding ghosts to WaterA/WaterB does not change the universe since the ghosts are already
    * in it.  We are just adding that ghost's index to WaterA/WaterB.
    *
    * Much of what is being discussed here is somewhat implementation details because in
    * a typical module, say Hartree-Fock, you are just going to use the Molecule you are
    * handed, you are not going to change it.
    *
    * Now we point out how to get say only the real atoms:
    */
    Molecule RealAtoms=SomeOtherMolecule.Fragment([](const Atom& AnAtom){return IsReal(AnAtom);});
    
    /* IsReal is a function that checks the atom for internal flags and returns true if its input
     * is not a ghost atom, dummy atom, or a point charge.  Similar functions: IsGhost, IsDummy, and
     * IsPointCharge exist.  As other atom types are added please update this statement.
     *
     * The last sort of common molecular manipulation that one may need to do is rotate/translate.
     * We provide free functions for that as well:
     */
     Molecule MovedMolecule=Reorient(WaterA,RotationMatrix,TranslationVector);
     
     /* Internally, Reorient, makes a new universe, copies each atom, repositions it, adds
      * that atom to the new universe, and then finally adds that universe to a new molecule,
      * which it returns.  Again, this ensures that the atoms inside WaterA do not change.
      * Note that any link between MovedMolecule and WaterA is broken.
      */
```


<!---

### Our Atom Class
   * Contains its basis set
     * Do ECPs go here? 
   * Contains a nucleus subclass tracking isotope data as well as finite nucleus data
     * Should atomic number be moved here?
     * Should mass be moved here?
   * Knows empirical physical data for that element, such as covalent/VDW raddii etc. (implemented via pointer to some const class)
   * The above are likely going to be the same for all atoms with the same atomic number, 
      * I therefore propose they are shared ptrs to const instances so they can be compared quickly by comparing addresses
      * The const-ness assures that you have to change what the pointer points to to modify the properties
      * Because Atoms are only returned by copy or const reference modifying these properties only modifies them in the copy which in turn can not be reinserted into the molecule because it is no longer in the universe within that molecule.
   * We neeed to discuss what equality of atoms means
     * I propose the following must be equal:
       * atomic number
       * charge
       * mass
       * number of electrons
       * multiplicity
       * coordinates
       * nucleus subclass
       * empirical physical data
       * basis set
     * In particular this ignores the index, because in my mind, it is a relative position in a universe used for quickly accessing a particular atom.
     * This definition would allow the following very useful scenario.  Let a and b be molecules with universes A and B respectively.  Now let A be a proper subset of B.  Even though b belongs to a different universe and thus its atoms may have different indices, operations like B+=A would still work
        * As universe/MathSet stand I don't think this would quite work because I think we assume the universe are the same for union.  Hence we would need to change checks to is subset (note if A==B a is still a subset of B (and vice versa)).
     
    
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
  
Arbitrary nested fragments, that is each fragment is a perfectly good molecule and can be fragmented as well
 Effeciency: memory for each atom is contigious allowing for BLAS calls on coordiantes, masses, charges, etc. This is done without duplicating data (basically there's some messy pointer redirection the user need not concern themselves with).
-->

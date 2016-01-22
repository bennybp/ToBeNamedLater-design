# PROP 0008 - Molecule

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Our chemical system of interest           |
| Status:        | In development                            |
 

## Purpose
 In chemistry we are interested in molecules and their environments and want to perform compuations on these "systems".  The system may contain multipole moments, implicit solvation (usually through multipole moments), electric fields, magnetic fields, extra basis functions, points of reference, etc.  Because the name "system" is confusing in a computer science context we will continue to call the main class molecule even though it is more than just atoms.
 
## Possible Heirarchy/ Implementation
There's really two ways we can go about doing this: define a general system that (usually) will have  lots of null properties or a hierarchy of systems.

Here's my take on a hierarchy of systems
![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class for an object that has set-like
*properties in a mathematical sense.  This
*class is templated on an element type T and
*a storage class U.  I can't use angle brackets
*with this UML so read TU as <T,U>.  Universe
*and StorageClass are actually shared pointers.
*/
class MathSet_T_U{
Vector_T Elements;
MathSet_T_U Universe;
U StorageClass;
}
/**
*@opt all
*@note A base class for a point, such as an atom or a shell
*carts are where the point is located, weights are properties
*associated with the point.
*/
class Point{
int index;
reference_vector Carts;
reference_map Weights;
}
/**
*@opt all
*@note An interface to a point reflecting its atom status
*weights include atomic number, mass, charge
*/
class Atom extends Point{}
/**
*@Opt all
*@note An interface to a point reflecting its shell status
*weights includes exponents and coefficients
*/
class Shell extends Point{
int NPrims;
int AngularMomentum;
}
/**
*@Opt all
*@note a class for storing all the data contigiously
*/
class Storage{
vector Carts;
map Weights;
}
/**
*@Opt all
*@note Implements a molecule by setting
*T=Atom and U=Storage
*/
class Locus extends MathSet_T_U{
}
/**
*@Opt all
*@note Implements a Gaussian basis set by setting
*T=Shell and U=Storage
*/
class BasisSet extends MathSet_T_U{
}
)

Within this class heirachy, fragments are simply subsets.  New systems can be made by taking unions.  Complements can be done
relative to a universe of discord.  The same machinery is used for both basis sets and molecules, with an interface specific to the actual object tacked on top.

## Features
 * Recursive fragments
 * Iteration over all atoms within a fragment, including the top-level "Molecule"
 * Iteration over all sub-fragments within a fragment (right now this requires a MathSet<MathSet<Atoms> >)

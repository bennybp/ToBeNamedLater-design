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
*@note A base class for a point, such as an atom
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
*@note a class for storing all the data
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

Common tasks when given a set of points are to compute the zero and first moment (center of mass and moments of inertia when mass is the weight, or the charge of the system and dipole moment if the atomic charges are used).  We avoid having to implement those features multiple times by creating the WeigthedPoint and Locus classes.  The WeightedPoint class actually supports many weights (can put mass, charge, MM parameters, etc. there).  The Atom interface seperates one call, GetWeight() of the WeightedPoint class, in to GetZ(), GetCharge(), etc.  We then further refine the interface of atom to ghost, point charges (and higher-order, point, multipole moments), molecular mechanics (MM) atoms, and dummy atoms.  Each of these imposes restrictions on the Atom base class, for example a ghost atom has no charge, no electrons, no mass, but does have an atomic number.  

Mathematically, a locus is the set of all points that satisfy some condition (e.g. the locus that is equidistant from a given
point, is a circle).  This condition may be as trivial as say "the locus in my molecule", which is my justification for using the name locus.  Either way, we have given the Locus class some basic set-theory members so that you can create larger locus from smaller locus.  The molecule interface refines the Locus interface so that the function calls are physically meaningful, e.g. Moment(0) with respect to mass becomes center of mass. 

Given a molecule it is common to want to fragment it (the fragmentation pattern may be defined by the user or via an algorithm). The goal is take one Molecule class and make it into many Molecule classes.  What we need is then a set of Molecule objects.  Note that this set of Molecule classess is not a Molecule itself (what we have is actually a set of sets of atoms, you have to "contract" in some manner to reduce to a molecule).  To me it is more natural to put this set of molecules in an STL container rather than making a class to hold all of these molecules.  Thus we extend the Molecule class to a Fragment class, which knows about its parent molecule, and can correspondingly interact with other Fragments.  The parent molecule provides us with a universe of discourse and thus allows for additional set operations.

TODO: How does symmetry fit into this?  At the Locus level?

## Features
 * Recursive fragments
 * Iteration over all atoms within a fragment, including the top-level "Molecule"
 * Iteration over all sub-fragments within a fragment

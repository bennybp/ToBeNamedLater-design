# PROP 0008 - Spatial Symmetry

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | How we plan on handeling spatial symmetry |
| Status:        | In development                            |
 

## Purpose
Although "real molecules don't have symmetry" for some molecules spatial symmetry can be used to provide savings.  Neglecting this cost savings is naive as it puts us at a disadvantage with respect to many other codes; however, explicity worrying about it in each routine means developers are likely to skip it as it is a major hassle on their part.  Presently we propose a library that will greatly automate much of this.  This is purely conceptual at the moment.

## Desired Features
* Should play nicely with the tensor object so that symmetry can be exploited in contractions
* Takes a set of Cartesian points, which are not necessarily the molecule, and some weights associated with those points
  * `MathSet<Point<Key,T>,PointStorage<Key,T> >` is the logical choice for what it should operate on
  * Molecules, grids, basis sets all derive from this class making symmetry applicable to them
* Non-Albeian group support (at the very least hooks for it should be in place)
  * I'm told it doesn't help in electronic structure theory, aside from geometry optimization, but it also really bothers me to a     priori discount something
* Translational symmetry in addition to point group symmetry 

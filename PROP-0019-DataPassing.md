# PROP 0019 - Data Passing

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | Protocol for passing data                 |
| Status:        | In development                            |


## Purpose

Under the module philosophy each module should be given all the input necessary for it to perform its task.  For example, an SCF module should be given a molecule, basis set, options (pertaining to convergence, number of iteratrions, etc.) and should then
perform an SCF.  For methods like SCF, the vast majority of the input will be contained within a class Wavefunction.  Methods should prioritize the use of data contained within the Wavefunction rather than say trying to compute their own MOs.  At the moment, Wavefunction is defined as:

![Alt text](http://g.gravizo.com/g?
/**
*@opt nodefontsize 14
*@hidden
*/
class UMLOptions{}
/**
*@opt all
*@note Class containg information pertaining to the approximate wavefunction
*/
class Wavefunction{
public SharedPointer Molecule;
public SharedPointer Basis;
public Tensor MOCoeffs;
public Tensor OrbitalEnergies;
public Tensor CIMRCoeffs;
}
)

The molecule and basis should be self explanatory, they are the molecule and basis set used to create your current wavefuntion.  The MOCoeffs are how to form the MOs from the AOs contained in the basis set.  The orbital energies are, well, the energies of the MOs.  The CIMRCoeffs are how the many-electron wavefunction is expanded in terms of Slater determinants, i.e. the configuration interactio or multi-reference coefficients (likely we will extend this Tensor to also hold coupled cluster amplitudes as they serve a similar purpose).

Food for thought.  Does it make sense to combine the Basis, MOCoeffs, and OrbitalEnergies into a class called Reference, which is the Slater determinant we are using as the reference?

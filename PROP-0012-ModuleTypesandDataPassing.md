# PROP 0012 - Module Types and Data Passing

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | A list of modules and their hierachy      |
| Status:        | In development                            |
 

## Purpose



![Alt text](http://g.gravizo.com/g?
  digraph G {
    aize ="4,4";
    node [shape=box]
    ModuleBase [label="BaseModule"];
    ModuleBase -> Method;
    OneInts [label="1eInts"];
    ModuleBase -> OneInts;
    TwoInts [label="2eInts"]
    ModuleBase -> TwoInts;
    ModuleBase -> CoreBuilder;
    OneInts -> CoreBuilder [style=dotted];
    TwoInts -> JKBuilder [style=dotted];
    JKBuilder -> GBuilder [style=dotted];
    ModuleBase -> JKBuilder;
    ModuleBase -> GBuilder;
    ModuleBase -> FockBuilder;
    CoreBuilder -> FockBuilder [style=dotted];
    GBuilder -> FockBuilder [style=dotted];
    Method -> SCF;
    FockBuilder -> SCF [style=dotted];
    ModuleBase -> DensityBuilder;
    DensityBuilder -> SCF [style=dotted];
    ModuleBase -> Properties;
    ModuleBase -> GuessBuilder;
    GuessBuilder -> SCF [style=dotted];
    ModuleBase -> UpdateGuess;
    UpdateGuess -> SCF [style=dotted];
    ModuleBase -> Localizer;
    ModuleBase -> IntTransforms;
    Method -> MP2;
    IntTransforms -> MP2 [style=dotted];
    SCF -> MP2 [style=dotted];
  }
)

- ModuleBase: Implements basic module behavior associated with using the module locator
- Method: Module capable of computing energy derivatives
  - Member function: Deriv which returns the i-th derivative.  Calls finite difference if it doesn't exist 
- 1eInts: One-electron integrals
  - Integrals choosen based on basis type
  - Input should be basis set and operator
- 2eInts: Two-electron integrals
  - Similar to 1 electron ints 
  - Density fitting lives here (?)
  - AO and MO versions (?)
    - Usually thought of as AO-like: (mu nu||lambda sigma), but MO-like (rs||tu) appear in correlated theories
- CoreBuilder: In charge of building the one-electron contribution to the Fock operator (uses 1eInts)
  - In additional to normal Fock core builds PCM, EFP, relativistic corrections, etc.
  - On account of latter, depends on density and should be inside the SCF iterations!!!!!
- JKBuilder: Builds J and K matrices (uses 2eInts)
  - Should be able to build just J, just K, or both
  - Multipole moments for J when electrons are "distant"
- GBuilder: In charge of building the two-electron contribution to the Fock operator (uses JKBuilder)
  - In charge of scaling J and K
- FockBuilder: In charge of building the Fock operator (uses CoreBuilder and GBuilder)
  - Just adds H and G (?)
  - Doer of Fock extrapolations (?)
  - Necessary (?)
- SCF: Performs both Hartree-Fock and density functional theory
  - Builds guess
  - Uses guess to build F
  - Updates guess
  - Checks for convergence
- DensityBuilder: Builds electronic densities
   - Computes "psi star psi" given two wavefunctions, which may be the same 
- Properties: Computes density based properties
  - Charges, ESP, MOs
- GuessBuilder: Builds the SCF starting guess
  - Core, SAD, Read
  - Is this just a special case of "updating a null guess"?
- UpdateGuess: Updates the SCF guess
  - DIIS, purification 
- Localizer: Localizes MOs
- IntTransforms: Transformations for 2eInts to MO basis
- MP2 performs second-order Moller-Plesset perturbation theory (N-th order ?)

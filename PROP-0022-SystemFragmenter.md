# PROP 0022 - System Fragmenter

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | The specifications of the system Fragmenter      |
| Status:        | In development                            |
 
## Purpose

A variety of methods rely on decomposing the system of interest into subsystems.  For example, QM/MM partitions the system into a
QM and and MM region, many-body expansions decompose a supersystem into monomers, dimers, etc., canonical SAPT requires decomposing
a dimer, and FSAPT requires identifying functional groups.  We will call these subsystems fragments for
the purposes of this proposal regardless of what they are normally called in a given instance, the supersystem will simply be
referred to as the system.

The purpose of the System Fragmenter module base is to abstract away the details of how the fragments are made so that the method
can concentrate on what to do with them.  This means to introduce a new way of fragmenting a system one only needs to implement a
new system fragmenter.

## Considerations
- Must take a system, which will be of the Pulsar type System
- Must return a set of fragments, each of which are of type System
  - The fragments and system need to be of the same type to ensure seemless interoperability
  - Fragments should have some memory of the system they came from
    - Handled for us by the System class's reliance on MathSet
- The fragments should have tags to facilitate organizing them, I term these tags serial numbers, although I kinda like frag tag
- In nearly all scenarios listed above the energy of the system is approximated by a linear combination of the fragment energies
  - It is somewhat novel to think of SAPT in this manner, but ultimately SAPT produces an energy that is the energy of the two
    fragments plus the current estimate of their interaction, i.e. it still involves a linear combination of the fragment
    energies
  - These coefficients are established by an inclusion/exclusion principle which is expensive (exponentional scaling) in general,
    but very cheap for select fragmentation schemes; however, one must know they have this scheme ahead of time
  - The system fragmenter ist thus in a privaliged position because it knows the scheme, hence it should return
    the coefficients
- For methods like the MBE the final set of fragments is formed by taking unions of the initial fragments
  - The ability to make these unions (and establish their linear expansion coefficients) should reside with the fragmenter as the
    that is what we have established is expected to return the coefficients
  - Note for BSSE computations the final set of fragments, hence the ones returned by the fragmenter, are obtained by taking 
    unions of ghost fragments and real fragments
    - This is thus a nesting of SystemFragmenters
  - Currently the set of unions to be made is controlled with an option "TRUNCATION_ORDER"
    - A thought is to make this a function parameter to the SystemFragmenter, again this involves the method in the fragmentation
      process, albiet to a lower extreme (they don't have to call additional functions)
      - Hence any method wanting to use this feature would now pick up the "TRUNCATION_ORDER" option
        - Based on the list above this would be the MBE, but may be needed for 3-body SAPT (?)
        - The fragmenter for VMFC and CP would also need this option, but could use the one passed in to it
          - These fragmenters are called from the MBE method meaning no additional methods pick up the truncation order
          - For these fragmenters it is a bit ambigous as to whether the input parameter should be for the ghosts or the reals
          
# APIs

The proposed return type of a SystemFragmenter is: `std::map<SerialNumberType,NMerInfo>` where SerialNumberType 
is: `std::set<std::string>` and NMerInfo is:

~~~{.cpp}
struct NMerInfo {
    SerialNumberType SN;//The serial number
    pulsar::system::System NMer;//The actual fragment
    double Weight;//The linear expansion coefficient
};
~~~

If each original fragment is tagged with a string, then inclusion of them in a C++ set allows an easy way to keep track of
which and how many original fragments made the final fragment as well as ensuring systematic unique tags for each of the final
fragments.  The double inclusion of the tag (once as the key and once within the struct) is to allow passsing only the struct to 
a function and not the entire map.

The purposed module API is then:

~~~{.cpp}
std::map<SerialNumberType,NMerInfo> SystemFragmenter::fragmentize(const pulsar::system::System& SuperSystem, size_t MaxUnions=1);
~~~

The return type was previously discussed.  The need to take the system as input is obvious.  The final parameter reflects the
maximum number of fragments that should be considered to form the final fragments.  The default argument of one reflects the
fact that really only the MBE would want something other than 1.


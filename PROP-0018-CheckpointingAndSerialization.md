# PROP 0018 - Checkpointing And Serialization

|                |                                                 |
|:---------------|:------------------------------------------------|
| Description:   | How we store data for restarts and communication|
| Status:        | Postponed                                       |
 

## Purpose
As everyone in the electronic structure community knows, jobs die.  Sometimes they die for reasons that can be overcome and in these situations one would like to be able restart the computation as close to the end as possible.  In order to do this the state of the computation needs to be saved, which we call checkpointing.  Within "the module project" the combination of the graph and the cache represent the state of the computation and backing them up is sufficient to allow restarts.  The easiest way to wrap up complicated objects is through serialization, which is why serialization is also part of this proposal.

## Serialization Features
Disclaimer: We will discuss serialization in terms of writing to a file, but it also applies to MPI communication as well.

In an object-oriented language such as C++, objects can be very complicated.  One way of storing them on disk is to write all of the member data to a file.  Then when one wants to read the object back in, a new instance is created and its member data is populated from the file.  This is fine in principle, but quickly becomes complicated and inefficient for modern class hierarchies, where one may not know exactly which class they have.  To oversimplify it a bit, serialization can be thought of as grabbing all the memory assoicated with the class and dumping it to disk.  This includes the member data, as well as the class structure, and (if done correctly) all child class member data and structures.  Reconstructing a class is simple because the memory has been snapshotted, one simply moves the class snapshot from disk to memory.  At the very least, serialization provides a uniform interface to storing disparate objects.

Boost::serialization is one of the standard libraries, but we are exploring other options to avoid Boost's bloat.  Ultimately, our serialization library must be capbable of:

* following common serialization conventions 
  * pass data to archive via `operator<<`
* capable of supporting arbitrary class hierarchies
* deep storing pointers
 
## Checkpointing features
The idea is that we serialize the graph and the cache.  The modulelocator will be rebuilt as it is trivial.  We then reexecute the graph starting with the first node.  Because each module stored their important, computationally expensive results like say the energy, in the cache, and because the input to each module has not changed when the module is called, the module looks in the cache, sees that it already knows the energy for this input and simply returns it.  For this to work it is absolutely essential that only the options that actually affect an outcome are used to "hash" an input.  That is changing the amount of memory or the number of threads doesn't (or at least it shouldn't) change the final SCF energy and thus shouldn't be used by the SCF module to cache a result.

Keeping track of what a module depends on is admittidly complicated so why not punt?  Each module defines a function:

```C++
bool SameOptions(/*Option set 1*/,/*Option set 2*/);
```

which returns true if the first set of input (could be molecule, basis set, and options objects, for example) generates the same output as the second set of input (made of the same classes).  Then, for example, upon restart the SCF module would ask each of its submodules if the input would change their result from the last computation, if so that module is rerun, otherwise, it just uses the previous result.



 

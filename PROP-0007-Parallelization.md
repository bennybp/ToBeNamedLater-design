# PROP 0007 - Parallelization

|                |                                           |
|:---------------|:------------------------------------------|
| Description:   | The top-level interface to parallelization|
| Status:        | In development                            |
 

## Purpose
Modern supercomputer capabilities can only be harnessed if your code is parallel.  This proposal is for a library that will provide a modern, user-friendly interface to a hybrid shared and distributed parallel environment in which most of these details are hidden from the user.  Instead, the user focuses on desiging and specifying "tasks" (basically functions that need run) and this library worries about how to run these tasks in a parallel fashion.  At the moment Ryan is wrapping Madness, which already provides much of this functionality.


## Features


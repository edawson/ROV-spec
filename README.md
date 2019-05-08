ROV-spec
---------------------
###The pROVenance data lineage format specification  
Eric T Dawson

## Background
Among other pitfalls, modern cloud analysis platforms for bioinformatics lack a
universal mechanism for reproducing analyses. The ROV format attempts to
succinctly yet completely describe the [Data
Lineage](https://en.wikipedia.org/wiki/Data_lineage) of a file in a user- and
machine-readable text format. From a single ROV file, a user should be able to
completely run and analysis assuming publicly available data and compute
resources.

## ROV basics
ROV is a tidy-translatable data format, meaning:
- The overall structure is implicitly a table where:
    - Every row is a single atomic observation
    - Every column is a single variable

However, columns are not always delimited by the same character, so they may
become compressed into what appears as a single column but is in fact multiple
variables. While not truly tidy, a series of string split operations and sorts
could produce a tidy format with only the information present in the table.


In addition to being tidy-translatable, ROV has the following tenets:
- ROV objects (lines of a ROV file) are immutable (i.e. cannot be modified later
  in the same description). This means ROV is purely functional.
- Input objects must be defined in a globally-accessible manner. This means that
  files on cloud storage receive the proper prefix (e.g. "gs://" for Google
  Cloud) and local files receive "file://" AND require a "SYS:Z:<TAG>" system
  name tag.
- Complex operations such as forks and joins are declarative (i.e. the commands
  to split and join used are clearly described).

## ROV's accidental data model
ROV is simply designed to be a record of work. However, a ROV file also
describes a complete computation graph when the following criteria are met:
1. All inputs are globally accessible
2. All functions (i.e. commands) are accessible from the runtime environment.
3. All parameters are defined.

When these conditions are met, ROV should perfectly model a computation graph
and its flow. This means it should be translatable to other workflow languages
such as WDL, CWL, nextflow, etc.

## Difference between ROV and Workflow Languages
ROV describes the full computational flow and the complete set of inputs /
outputs. In this sense, it is akin to a workflow language combined with the
input descriptors. However, ROV requires much stronger description of inputs /
intermediate files / outputs. ROV is also intended to be written programatically
for a single run, whereas workflow scripts are written by hand and run
programatically on many inputs.

## ROV types
A full ROV description involves several types. Each type is defined
by a line and describes an aspect of an analysis that is essential
to its reproduction.

### Reserved tags
ROV supports SAM-style tags (i.e. char keys, a type, and an arbitrary
value). Some special keys are reserved by ROV.

- CT:S:\<val\> : file creation time.
- DI:S:\<val\> : digest (default: md5) of an input, parameter, or executable
- SYS:S:\<val\> : system name tag

### Input types
Files are immutable, meaning outputs can be returned but files can't
be overwritten in the platform. All inputs must be defined globally,
meaning:  
- They are prefixed with their storage medium (e.g. "gs://" for GCE or
"file://" for local storage)
- If files are local, the system name is tagged.
- An md5sum digest is included with every input.

### Parameter types
Parameters represent inputs that are not stored on a fileystem, and therefore
do not have the same global-accessability requirements. They may be integers,
floating point numbers, or strings

Parameters can also be defined in an array type (a string) which is passed
to a Function call. In addition, they may be encapsulated within the
command line of a function call

### Environment types
An environment is both a physical system or cloud platform and a runtime
environment. The environment may be a docker container, a virtual machine
image, a singularity container, or a set of required local compute commands
with corresponding versions.

Certain information is required 

### Execution types
An execution defines a command line that takes in inputs and parameters (inside an environment)
and (optionally) produces outputs.

### Output types
An output describes a file or vale output by a function. Like inputs,
they must have their global location defined and receive an md5sum.

### Converting between WDL and ROV
WDL (as well as many other workflow description languages) define similar concepts to ROV. Given a set of inputs and 
environment definitions, valid ROV can be generated from WDL. The same is likely true of other languages, although we have not implemented such handling yet.


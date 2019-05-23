ROV-spec
---------------------

**A pROVenance / data lineage format specification**  
v0.1.0  
Eric T. Dawson

## Background
Among other pitfalls, modern cloud analysis platforms for bioinformatics lack a
universal mechanism for reproducing analyses. The ROV format attempts to
succinctly yet completely describe the [Data
Lineage](https://en.wikipedia.org/wiki/Data_lineage) of an output file in a user- and
machine-readable text format. From a single ROV file, a user should be able to
completely run an analysis, assuming they have access to the data and compute
resources.

## ROV as a record of work
Log files are often dense and not human readable. In addition, they often do not capture
enough information to fully reproduce an analysis. ROV attempts to address this with a simple,
human readable format that can contain sufficient information for reproducing an analysis. Inputs/outputs
must be defined by both their name ("ID") and contents (md5sum). Using strict versioning (e.g. docker image tags,
CPU architecture information, system hostname, date), the execution environments of ROV should be reproducible as well.
While other formats, such as the [Biocompute Object project from FDA](https://github.com/biocompute-objects/BCO_Specification)
have sought to address this, these formats have proved unwieldy. ROV attempts to contain only what is necessary and sufficient
for a given workflow to produce defined outputs from defined inputs. ROV explicitly avoids any information about the user,
as it is our belief that knowing the user should not be a prerequisite for a successful workflow.

## ROV as a DSL for universal computing
A convenient byproduct of ROV's design is that it can function as both a record of work
and a reproducible analysis script. ROV enables anyone with a sufficiently-defined ROV file
and access to the underlying resources to run and validate a task. When inputs and
compute environments are public (e.g. public data and Docker images), this means anyone should be able
to reproduce the original work.


## ROV schema
ROV is a tab-separated, line-delimited format. Each line describes an *occurrence*
of a specific *type*. There are fix basic ROV types, which provide an abstraction for
describing the necessary units of a task:  

1. I / input : A file, which is defined with a universal-style URL-like identifier.
   These become inputs to tasks.
2. P / parameter : Non-file inputs to tasks (e.g. integers, strings, environment variables, etc)
3. E / environment : A strict description of the compute environment used, including:
    - a SYS tag, describing the system that ran the command.
    - other information that defines the compute environment (e.g. CPUs, system name, etc)
4. X / execution : Describes the command line that is run and its dependant inputs / parameters.
5. O / outputs: Describes the outputs of an executed task.

These types completely mirror the types of the [GA4GH TES API](https://github.com/ga4gh/task-execution-schemas).
In addition, they represent an independent computation graph. ROV should perfectly model most modern workflow languages
(e.g. WDL, CWL, nextflow, etc) and mapping between the two (plus configured inputs / parameters) should be relatively simple.

Further descriptions of the individual line types follow.

## Future additions
In the near future we aim to support two new lines (F / fork and J / join) that provide abstractions for splitting/sharding inputs
and recombining intermediate files. These are analagous to WDL's scatter-gather format, though they may also be used to describe
UNIX split/cat operations, or domain-specific splits/joins such as splitting a BAM file by chromosome.

### Input types
Files are immutable, meaning outputs can be returned but files can't
be overwritten in the platform. All inputs must be defined globally,
meaning:  
- They are prefixed with their storage medium (e.g. "gs://" for GCE or
"file://" for local storage)
- If files are local, the system name is tagged. For many users within organizations, this is sufficient to identify them.
If external use is required, we encourage defining a globally unique system name (e.g. NIH-Biowulf)
- An md5sum digest is included with every input and output


### Parameter types
Parameters represent inputs that are not stored on a fileystem, and therefore
do not have the same global-accessability requirements. They may be integers,
floating point numbers, or strings

Parameters can also be defined in an array type (a string) which is passed
to a Function call. In addition, they may be encapsulated within the
command line of a function call, although we don't recommend this practice.

We discourage environment variables as parameters, as they are easily hidden.
However, they may be either included as P lines or set up in the Environment
defined by an E line.

### Environment types
An environment is both a physical system / cloud platform and a runtime
environment. The environment may be a docker container, a virtual machine
image, a singularity container, or a set of required local compute commands
with corresponding versions.

While ROV doesn't explicitly require 

### Execution types
An execution defines a command line that takes in inputs and parameters (inside an environment)
and (optionally) produces outputs.

### Output types
An output describes a file or vale output by a function. Like inputs,
they must have their global location defined and receive an md5sum.


## ROV theory
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
- Some ROV types (e.g. X / executable lines) require specific tags to maintain
  universal operability.
- Complex operations such as forks and joins are declarative (i.e. the commands
  to split and join used are clearly described).

## ROV as a computation graph
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

### Reserved tags
ROV supports SAM-style tags (i.e. char keys, a type, and an arbitrary
value). Some special keys are reserved by ROV.

- CT:S:\<val\> : file creation time.
- DI:S:\<val\> : digest (default: md5) of an input, parameter, or executable
- SYS:S:\<val\> : system name tag

### Converting between WDL and ROV
WDL (as well as many other workflow description languages) define similar concepts to ROV. Given a set of inputs and 
environment definitions, valid ROV can be generated from WDL. The same is likely true of other languages, although we have not implemented such handling yet.


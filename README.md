# Data Versioning #

Version control is a huge part of reproducible research and open source software development. Versioning provides a complete history of some digital object (e.g., a software program, a research project, etc.) and, importantly, allows one to trace what changes have made to that object, when those changes were made, and (with the appropriate metadata) why those changes were made. This document holds some of my current thinking about version control for data.

## Background Context ##

Especially in the social sciences, researchers depend on large, public datasets (e.g., [Polity](http://www.systemicpeace.org/polity/polity4.htm), [Quality of Government](http://qog.pol.gu.se/), [Correlates of War](http://www.correlatesofwar.org/), [ANES](http://electionstudies.org/), [ESS](http://www.europeansocialsurvey.org/), etc.) as source material for quantitative research. These datasets typically evolve (new data is added over time, corrections are made to data values, etc.) and new releases are periodically made public. Sometimes these data are complex collaborative efforts (see, for example, Quality of Government) and others are public releases of single-institution data collection efforts (e.g., ANES). While collaborative datasets create a more obvious use case for version control, single-institution datasets might also be improved by version control. This is particularly important because old releases of these vital datasets are often not archived (e.g., ANES) meaninig that it is essentially impossible to recover a prior version of a given ANES dataset after a new release has occurred. This post is meant to steer thinking about how to manage the creation, curation, revision, and dissemination of these kinds of datasets. While the ideas here might also apply to how one thinks about managing their own data, they probably apply more at the stage of data creation than at later data use after a dataset is essentially complete or frozen.

## Review of Existing Tools and Approaches ##

The Open Data Institute has [a nice post](https://theodi.org/blog/adapting-git-simple-data) outlining the challenges of using standard, software-oriented version control software (namely, git) for version control of data. The main issue is that git, like almost all VCS, is designed to monitor changes to lines in text files. This makes a lot of sense for code, as well as for articles, text, etc. But it starts to make less sense for digital objects where a line is not a meaningful unit. This becomes really clear when we start to version something like a comma-separated values (CSV) file (as the ODI post describes). Changing a single data field leads to a full-line change, even though only one cell actually changed. A similar problem emerges in XML, JSON, or other text-delimited formats (though, note that [the Open Knowledge Foundation seems to favor JSON as a storage mode](http://dataprotocols.org/data-packages/)). 

Using the linebreak `\n` as delimiter to track changes to an object does work for data. [daff](https://github.com/paulfitz/daff) is a javascript tool that tries to make up for that weakness. It provides a diff (comparison of two file versions) that respects the tabular structure of many data files by highlighting cell-specific changes. This is not a full version control system, though, it's simply a diff tool that can serve as a useful add-on to git.

[A post on the Open Knowledge Foundation blog](http://blog.okfn.org/2013/07/02/git-and-github-for-data/) argues that there is some logic to line-based changesets because a standard CSV (or any tabular data file) typically records a single observation on its own line. Line-based version control then makes sense for recording changes to observations (thus privileging rows/obervations over columns/variables).

[dat](http://dat-data.com/about.html) aims to try to be a "git for data" but the status of the project is a little unclear, given there hasn't been very active [development on Github](https://github.com/maxogden/dat). [A Q/A on the Open Data StackExchange](http://opendata.stackexchange.com/questions/748/is-there-a-git-for-data) points to some further resources for data-appropriate git alternatives, but there's nothing comprehensive.

[Universal Numeric Fingerprint (UNF)](http://guides.dataverse.org/en/latest/developers/unf/index.html) provides a potential strategy for version control of data. Indeed, that's specifically what it was designed to do. It's not a version control system per se, but it provides a hash that can be useful for determining when a dataset has changed. It has some nice features:
  - File format independent (so better than an MD5)
  - Column order independent (a dataset where columns have been shuffled produces the same UNF hash)
  - Consistently handles floating point numbers (rounding and precision issues are unproblematic)
But, UNF is not perfect. The problems include:
  - It is sort-sensitive (an identical dataset that is row sorted produces a different UNF; thus privileging columns/variables over rows/observations)
  - There is no handling of metadata (e.g., variable names are not part of the hash)
  - It is quite sensitive to data structure (e.g., "wide" and "long" representations of the same dataset produce different UNFs)
  - It is not a version control system and provides essentially no insights into what changed, only that a change occurred

All of these tools also focus on the data themselves, rather than associated metadata (e.g., the codebook describing the data). While some data formats (e.g., proprietary formats like Stata's .dta and SPSS's .sav) encode this metadata directly in the file, it is not a common feature of widely text-delimited data structures. Sometimes codebooks are modified independent of data values and vice versa, but it's rather to see large public datasets provide detailed information about changes to either the data or the codebook, except in occasional releases.

Another major challenge to data versioning is that existing tools version control are not well-designed to handle provenance. When data is generated, stored, or modified, a software-oriented version control system has no obvious mechanism for recording *why* values in a dataset are what they are or why changes are made to particular values. A commit message might provide this information, but as soon as a value is changed again, the history of changes *to a particular value* are lost in the broader history of the data file as a whole.

So, there are clearly no complete tools in existence for versioning data.

## Some Principles of Data Versioning ##

The first principle of data versioning is that changes to data have sources or *explanations*. A system of data versioning must be able to connect data values, structure, and metadata (and changes to those features) to explanations of those values or the changes to values at the value level (rather than at the level of variables, observations, or files).

The second principle is that data versioning should be value-based, not variable- or observation-based. A system cannot privilege observations over variables or variables over observations; a change to an observation is necessarily a change to a variable, and vice versa.

The third principle of data versioning is that data exist independent of their format. If one changes a data value in a CSV versus a JSON tree, those are content-equivalent changes. As such, any system of version control should allow data users to interact with data in whatever file format they choose without necessarily using the underlying data storage format.

The fourth principle of data versioning is that collaboration is essential to data generation and curation. A system of data versioning must be natively collaborative and logically record who is generating and modifying data.

The fifth principle of data versioning is that changes to data structure should be recorded independently of data values. Sort order of observations, the arrangement of columns/variables, and the arrangement of rows as cases versus case-years, etc. (i.e., "wide" versus "long" arrangements) are structural features of a dataset, not features of the data per se. These are important, but a data versioning process should be able to distinguish a change to the content of a dataset from a change to the organization or structure of a dataset and, in the latter case, correctly recognize as identical a dataset that is arranged in two different ways.

The sixth principle of data versioning is that metadata matters but, like structure, should be handled separately from changes to data. Two identical datasets with different metadata should be recognized as content-identical.

## Tentative Conclusions ##

Data should be stored in a [key-value](http://en.wikipedia.org/wiki/Associative_array) manner, where an arbitrary key holds a particular data value. A mapping then connects those particular data values to both observations and variables, so that any assessment of changes to data are format-independent and structure-independent. As such, a change to a value is recorded first as a change to a value but can be secondarily recognized as a simultaneous change to both an observation and a variable.

Any interface to such a key-value store should come in a familiar and flexible form: users should interact with a data versioning system via whatever manner they currently use data (e.g., a text editor, data analysis software, a spreadsheet application, etc.). Changes should be recorded in a master file that can natively and immediately import from and export to any data file format (including delimited files, spreadsheets, XML, JSON, etc.).

Data versioning systems must have a more sophisticated commit system than that provided by current, software-oriented version control systems. In particular, commits should record not only the change to a data value, structure, or metadata but also *structured* information that explains that change, including the reason for the change, the source(s) of the data value, and the author of the change. In essence, both changesets and states of the complete data should be fully citable and carry citation-relevant metadata.

---
(c) 2015 Thomas J. Leeper, [released under CC-BY](https://creativecommons.org/licenses/by/2.0/)

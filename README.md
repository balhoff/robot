# ROBOT is an OBO Tool

[![Build Status](https://travis-ci.org/ontodev/robot.svg?branch=master)](https://travis-ci.org/ontodev/robot)

ROBOT is a tool for working with [Open Biomedical Ontologies](http://obofoundry.org). It's still in development!


# Usage

This code can be used as:

1. a command-line tool
2. a library for any JVM language


## 1. Command Line Tool

The command-line tool is packaged a Java JAR file and can be run via the `robot` shell script.

1. Make sure that you have [Java 7 or later installed](https://www.java.com/en/download/installed.jsp)
2. Download **two** files: the [`robot.jar`](https://build.berkeleybop.org/job/robot/lastSuccessfulBuild/artifact/bin/robot.jar) file (about 25MB) and the right script for your platform:
    - [`robot`](https://build.berkeleybop.org/job/robot/lastSuccessfulBuild/artifact/bin/robot) shell script for Unix, Linux, and Mac OS X
    - [`robot.bat`](https://build.berkeleybop.org/job/robot/lastSuccessfulBuild/artifact/bin/robot.bat) batch script for Windows
3. Put both files on your [system PATH](https://en.wikipedia.org/wiki/PATH_(variable)):
    - on Unix, Linux, and Mac OS X this could be `/usr/local/bin/`
    - on Windows this could be `C:\Windows\`
    - or update your PATH to include your chosen directory
    - NOTE: both files must be in the same directory
4. Now you should be able to run ROBOT from a command line:

        robot help

See [examples/README.md](https://github.com/ontodev/robot/tree/master/examples/README.md) for a tutorial with many example commands.


## 2. Library

The core ROBOT operations are written in plain Java code, and can be used from any language that uses the Java Virtual Machine. The command-line tool is both built on top of the library of operations. You can add use these operations in your own code, or add new operations written in any JVM language.

You can download the latest [`robot.jar`](https://build.berkeleybop.org/job/robot/lastSuccessfulBuild/artifact/bin/robot.jar) file (about 25MB) to include in your projects.


### Java

The library provides a number of Operation classes for manipulating ontologies. The `IOHelper` class contains convenient methods for loading and saving ontologies, and for loading sets of term IRIs. Here's an example of extracting a "core" subset from an ontology:

    IOHelper ioHelper = new IOHelper();
    OWLOntology full = ioHelper.loadOntology("ontology.owl");
    Set<IRI> coreTerms = ioHelper.loadTerms("coreTerms.txt");
    OWLOntology core = ExtractOperation.extract(full, coreTerms);
    ioHelper.saveOntology(core, "core.owl");

Alternatively:

    IOHelper ioHelper = new IOHelper();
    ioHelper.saveOntology(
      ExtractOperation.extract(
        ioHelper.loadOntology("ontology.owl"),
        ioHelper.loadTerms("coreTerms.txt"),
        IRI.create("http://example.com")
      ),
      'core.owl'
    );


# Build

We use [Maven](http://maven.apache.org) as our build tool. Make sure it's [installed](http://maven.apache.org/download.cgi), then run:

    mvn clean package

This will create a self-contained Jar file in `bin/robot.jar`.

Other build options:

- `mvn clean test` runs JUnit tests with reports in `[module]/target/surefire-reports`
- `mvn clean verify` rebuilds the package and runs integration tests against it, with reports in `[module]/target/failsafe-reports`
- `mvn site` generates reports (including Javadoc and Checkstyle) in `target/site` and `[module]/target/site`


## Development

We use [Checkstyle](http://checkstyle.sourceforge.net) to enforce the default [Sun Guidelines](http://www.oracle.com/technetwork/java/javase/documentation/codeconvtoc-136057.html), with some exceptions in [checkstyle-suppressions.xml](checkstyle-suppressions.xml). Run `mvn clean site` to generate the Checkstyle reports in `target/site/checkstyle-aggregate.html`. Some of the rules are admittedly arbitrary, but consistency should help newcomers read the code. See also the [.editorconfig](.editorconfig) file, which can provide some [hints to your text editor](http://editorconfig.org).

Basics: indent with four spaces; maximum line length is 80 characters; use spaces after commas, before and after operators such as `+`; no trailing whitespace; UTF-8 encoding; Unix newlines; remove unused imports; always include JavaDocs.


# Design

The library provides a set of Operations and a set of Commands. Commands handle the command-line interface and IO tasks, while Operations focus on manipulating ontologies. Sometimes you will have the pair of an Operation and a Command, but there's no necessity for a one-to-one correspondence between them.

Commands implement the Command interface, which requires a `main(String[] args)` method. Each command can be called via `main`, but the CommandLineInterface class provides a single entry point for selecting between all the available commands. While each Command can run independently, there are shared conventions for command-line options such as `--input`, `--prefix`, `--output`, etc. These shared conventions are implemented in the CommandLineHelper utility class. There is also an IOHelper class providing convenient methods for loading and saving ontologies and lists of terms. A simple Command will consist of a few CommandLineHelper calls to determine arguments, a few IOHelper calls to load or save files, and one call to the appropriate Operation.

Operations are currently implemented with static methods and no shared interface. They should not contain IO or CLI code.

The current implementation is modular but not pluggable. In particular, the CommandLineInterface class depends on a hard-coded list of Commands.


## Term Lists

Many Operations require lists of terms. The IOHelper class defines methods for collecting lists of terms from strings and files, and returning a `Set<IRI>`. Our convention is that a term list is a space-separated list of IRIs or CURIEs with optional comments. The "#" character and everything to the end of the line is ignored. Note that a "#" must start the line or be preceded by whitespace -- a "#" inside an IRI does not start a comment.

# Acknowledgments

The initial version of ROBOT was developed by James A. Overton, based on requirements and designs given by Chris Mungall, Heiko Dietze and David Osumi-Sutherland. This initial version was funded by P41 grant 5P41HG002273-09 to the Gene Ontology Consortium.

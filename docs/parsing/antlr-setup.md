# Setting up ANTLRv4

For parsing a DFM file with our grammar, we will first need to set up *ANTLR*, its classpath, and some aliases (optional, but heavily recommended).

## Prerequisites

- [Java JDK](https://www.oracle.com/java/technologies/downloads/#jdk26-windows) (latest but earlier versions should work) 
- [ANTLRv4 complete jar](https://www.antlr.org/download.html)

*ANTLR* can be manually downloaded from the source above, or using the command line tool `curl` to grab it:

```bash
$ cd /usr/local/lib
# check latest version (at the time of writing this documentation 4.13.2 is the latest)
$ curl -O http://www.antlr.org/download/antlr-4.13.2-complete.jar
```

## Instructions

1. Copy the downloaded tool where usually third-party java libraries live (up to preference); Example: `/usr/local/lib` or `C:\Program Files\Java\libs`
2. Add tool to `CLASSPATH` and to startup script (Example: `.bash_profile`)
3. Optional: also add aliases to startup script to simplify the usage of *ANTLR*

#### Linux/MacOS

```bash
# 1.
sudo cp antlr-4.x-complete.jar /usr/local/lib/
# 2. and 3.
# add this to .bash_profile
export CLASSPATH=".:/usr/local/lib/antlr-4.x-complete.jar:$CLASSPATH"
# simplify the use of the tool to generate lexer and parser
alias antlr4='java -jar /usr/local/lib/antlr-4.x-complete.jar'
# simplify the use of the tool to test the generated code
alias grun='java org.antlr.v4.gui.TestRig'
# set up alias for java compiler
alias javac='javac -cp ".:/usr/local/lib/antlr-4.x-complete.jar"'
```

#### Windows

```bash
# 1. Copy antlr-4.x-complete.jar in C:\Program Files\Java\libs (or wherever preferred)
# 2. Append the location of ANTLR to the CLASSPATH variable, or create a CLASSPATH variable if haven't already done so
# by pressing WIN + R and typing sysdm.cpl, then selecting Advanced (tab) > Environment variables > System Variables
# CLASSPATH -> .;C:\Program Files\Java\libs\antlr-4.x-complete.jar;%CLASSPATH%
# 3. Add aliases
# create antlr4.bat  
java org.antlr.v4.Tool %* 
# create grun.bat  
java org.antlr.v4.gui.TestRig %*
# put them in the system PATH or any of the directories included in PATH
```

Replace `4.x` with the downloaded version.

**Notice**: we will be assuming that the reader is using the same aliases as the ones specified here.

In the following section, we will utilize our refined grammar to parse a `.dfm` test file. We will show how the generated parser is built up, and output some tokens.
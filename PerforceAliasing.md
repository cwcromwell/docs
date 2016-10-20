# Server/2016.1/DVCS/CommandAliasing - P4


## Contents

1 Introduction

1.1 Why are we doing this?

2 Overview

2.1 Alias processing

2.2 What is an alias?

2.3 Adding an alias

3 Perforce Aliases by Example

3.1 I am a lousy typist

3.1.1 I make a lot of typos

3.1.2 I don't like typing long words

3.1.3 I can't remember who I am

3.2 I want to pretend I'm using git/subversion/etc

3.3 I want to alias my aliases

3.4 I speak my own language

3.5 I like my Format Better Than Yours

3.6 I Like Different Defaults

3.7 Simple System Administration

3.8 Aliases can have named arguments

3.8.1 Cherry picking

3.8.1.1 You can be explicit about your arguments

3.8.2 Job 78277 Simpler clone syntax

3.9 Manipulating input and output

3.9.1 Saving command output

3.9.2 Simple pipelining

3.9.2.1 job 78918 mergedown/copyup pipelines

3.9.3 Manipulating spec output

3.9.4 String substitutions

3.10 Putting it all together

3.10.1 Reopen the files in a certain changelist

3.10.2 Make a new task stream

3.10.3 deletion

3.10.4 Predecessors

3.10.5 Augmenting status

3.11 Aliases can communicate to multiple servers

4 More Formal Definition

4.1 Aliases File

4.1.1 Comments and whitespace

4.2 Syntax and Terminology

4.2.1 Command Structure

4.2.1.1 command

4.2.1.2 client-options

4.2.1.3 command-options

4.2.1.4 command-arguments

4.2.1.5 Example

4.2.2 Alias Structure

4.2.2.1 Alias Pattern

4.2.2.1.1 Enclose arguments in dollar-parens

4.2.2.2 Alias Transformation

4.2.2.2.1 Muli-command chains

4.2.2.2.2 Input/Output Redirection

4.2.2.2.3 Multi-line aliases

4.2.2.2.4 Variable Substitutions

4.2.2.2.5 Special Operators

4.3 Variables

4.4 Special Operators

4.4.1 p4subst

5 Practicalities

5.1 Per-workspace aliases files

5.2 Managing your aliases file

5.3 Developing your alias

6 Advanced topics

6.1 Multi-command chains

7 Really advanced stuff that probably won't make the first release

8 A bunch of links

## Introduction

For the 2016.1 release, we're proposing to build a new aliasing facility into the 'p4' command line
executable.

Here are a few very high-level things about this new functionality:

1. It's completely client-side, and will work with any server
2. It only works in the true command line version of p4, not in the derived clients, APIs, or GUIs.
3. It is not a full-featured scripting or programming language. We already have that: see P4Perl,
P4Python, P4Ruby, P4Java, etc.

The goal of this feature is to try to find the sweet spot which is more than you can currently do with simple
Bash aliases, but doesn't require that we bloat up the command-line with a complete programming
language embedded into it.

Although this project is being done by the DVCS team, and is expected to be useful primarily for DVCS
users, it is a general-purpose feature that will be part of the base command line, and so can be used by all
Perforce users.

### Why are we doing this?

One of the things we hear all the time is:

```
git has aliases. Why don't you guys have aliases?
```

To this, we currently have two answers, and will soon have a third:

1. git's aliases are one of the reasons git is very hard to use, particularly on Windows.

2. Perforce has multiple full-featured scripting APIs already (P4Perl, etc.)

3. OK, we will have aliases. We agree they can be very useful.

## Overview

You can define a set of command aliases. Typically, these are read from the .p4alias file in your home
directory (p4aliases.txt on Windows); later in this document we'll describe details about the syntax of this
file, as well as the P4ALIASES configuration setting.

At the very start of operation, before running your command, p4 will now perform alias interpretation; if
the command is modified due to one or more aliases, p4 will run the modified command instead.

Note that this is completely a client-side feature, and hence it works with any server/proxy/broker/replica
configuration that you have. The precise commands you can run, of course, still depend on the particular
server you're talking to.

Also, note that this is purely a command line client feature, not part of the API, so it only works in the
standard p4 command line client, not any derived clients or guis.

### Alias processing
Once p4 has figured out which command aliases (if any) have been defined, it processes them in order,
going through each alias until it finds one which describes a modification of the current command.
If it doesn't find any such alias, it is done, and it runs the command.

But if it does find such an alias, it performs the modification and then restarts from the beginning,
re-checking each alias.

This means that a command may end up being transformed 0, 1, or even multiple times before it is run,
depending on what particular aliases are present.

### What is an alias?

An alias consists of two parts:

*a pattern, which names the alias, and

*a transformation rule, which describes how to transform the command

In your aliases file, an alias is defined on a single line, with an equals sign after the pattern.

###Adding an alias

You can just edit your aliases file with your favorite editor.

We are considering also providing a simple 'p4 alias' command, so that you can do:

```
p4 alias ci = submit
```

and it will add (or replace) the 'ci' alias in your aliases file.

## Perforce Aliases by Example

One way to get comfortable with what our aliasing facilities provide is to look at a lot of examples.

So here are a lot of examples, grouped into categories, and sort-of arranged in increasing level of
complexity:

### I am a lousy typist

Some people like to use aliases because they can't type.

#### I make a lot of typos

```
clinet = client
snyc = sync
sumbit = submit
recnocile = reconcile
```

#### I don't like typing long words

```
itg = integrate
pop = populate
rc = reconcile
di = diff
```

I can't remember who I am

```
me = set P4USER
```

#### I want to pretend I'm using git/subversion/etc

Some people like to set up aliases to ease the transition between tools, whether it be git, cvs, svn,
whatever...

```
checkout = sync
commit = submit
purge = clean
stash = shelve
blame = annotate
stash-list = changes -s shelved
pull = fetch -r origin
```

#### I speak my own language

There are definitely common idioms, commands you type very frequently.

```
last = changes -m 1
last-dev = last //depot/dev/...
```

#### I Like Different Defaults

Some people feel the default flags were not chosen correctly.

```
annotate = annotate -u
grep = grep -i
my-changes = changes -u $(P4USER)
```

Note that the 'my-changes' alias (which is essentially equivalent to 2016.1's "changes --me") uses a variable
expansion (more about that below).

###Simple System Administration

Other system administration examples:

```
halt = admin shutdown
who-is-active = changes -m 3 && monitor show && lockstat
```

### Aliases can have named arguments

```
recent-changes $(max) = changes -m $(max)
recent-by-user $(u) $(m) = changes -m $(m) -u $(u)
my-recent-changes $(max) = changes -u $(P4USER) -m $(max)
``` 

```
kill-shelf $(cl) = shelve -d -c $(cl) && change -d $(cl)
```

(I'm always forgetting that second step, and leaving empty pending changeslists around. Now I won't.)

### One-shot branch deletion

Here's a fairly crude implementation of job 77862 (http://computer.perforce.com:8080/job077862?ac=111)
:

```
nuke-stream $(branch) = stream -d //stream/$(branch) &&
obliterate -y //stream/$(branch)/...
```

#### Augmenting status

As requested by job079803 (http://computer.perforce.com:8080/@md=d&cd=//&c=Zya@
/job079803?ac=111) , a more personalized 'status' command, that also shows me files that I need to sync:

```
my-status = status && sync -n
```

## More Formal Definition

Trying to strike the right balance between precision and readability...

### Aliases File

Your aliases, if you choose to define them, are stored in a single file called the aliases file.

The name and location of the aliases file uses the same basic mechanism as we use for the tickets file and
the trust file:

1. The file is named ".p4aliases" on Unix and Mac systems, and "p4aliases.txt" on Windows systems.

2. The file is stored in your $HOME directory on Unix and Mac systems, and in your $USERPROFILE
directory on Windows systems.

3. You can override the above two rules by specifying the full name and location of your aliases file in
the P4ALIASES environment variable.

#### Comments and whitespace

Blank lines in an alias file are quietly ignored.

If the first non-blank character in a line is '#', the entire line is ignored.

This allows you to embed comments in your alias file, and generally make it look fairly nice and readable.

### Syntax and Terminology

#### Command Structure

p4 commands have the following overall syntactical structure:

```
p4 [client-options] <command> [command-options] [command-arguments]
```

**command**

The command is a word like "sync" or "submit" or "integrate" or "revert".

**client-options**

The client options, if any, come before the command, and modify the behavior of the p4 client itself.

Examples include '-p' to set the P4PORT, and '-d' to set the logical working directory.

**command-options**

The command options, if any, come after the command, and are different for each command.

**command-arguments**

The command arguments, if any, come after the command options, and again the interpretation is up to
each command.

**Example**

In the command

```
p4 -ztag -c bpendleton-admin unload -c bpendleton-dev
```

the command is 'unload', the client-options are '-ztag -c bpendleton-admin', and the command-options are
'-c bpendleton-dev'.

### Alias Structure

Each alias definition has the following overall syntactical structure:

```
pattern = transformation
```

**Alias Pattern**

The alias pattern describes how we will know that a particular command is using this alias, and has the
syntax:

```
name [args...]
```

The simplest pattern is a single literal word, which names the alias, and is matched exactly against the
command the user issues.

A pattern can also specify 1 or more named arguments, in which case the alias name is matched exactly
against the command the user issues and the alias arguments are matched, positionally, against the "words"
the user provided after the command.

**Enclose arguments in dollar-parens**

Arguments for an alias are specified using the syntax '$(variable)', where 'variable' is a name that you pick.

You can have as many arguments as you want, although since they are matched positionally, you'll rarely want to use 
more than two or three at the most for a single alias.

Alias Transformation


The alias transformation rule describes what to do when we match the user's command with this particular
alias. The transformation rule can be simple or complex.

A simple transformation rule describes how to replace the user's command (and optional arguments) with a
replacement command (and optional arguments).

Complex transformation rules include the definition of:

1. multi-command chains

2. input and/or output redirection

3. multi-line aliases

4. variable substitutions

5. special operator (substitution and enumeration) usage

**Muli-command chains**

An alias transformation may expand to multiple commands by separating the individual commands with
the special token '&&'.

**Input/Output Redirection**

An alias transformation may redirect the input of a command using '<', and may redirect the output of a
command using '>'.

**Multi-line aliases**

Since the alias can be very long, it can be split across multiple lines by splitting it immediately after any
'&&' token in the transformation, or by splitting it at any word in the transformation and ending the line
with '\', so:

```
my-command = this &&
that &&
the other \
thing
```

**Variable Substitutions**

Variables in an alias transformation are referenced using the syntax '$(variable)', and have the appropriate
values substituted when the command is run.

**Special Operators**

There are certain special operators which can be used in constructing more complex command pipelines.

### Variables

There are several types of variable substitution:

1. The well-known P4USER, P4CLIENT, P4PORT, P4LANGUAGE, CWD, and OS variables are
directly recognized and substituted.

2. Variables from the alias pattern are substituted in the transformed command using the values
provided by the user on the command line

3. Input/Output redirection variables are automatically used to supply and retain, respectively, the input
and output of the command being run.

### Special Operators

This section is just a draft at this point.

We are contemplating providing certain special operators for certain common tasks:

1. simple string substitutions

2. manipulating spec data in more complex ways than string substitutions

3. iterating over lists (e.g., lists of files) -- right now, we think 'p4 -x -' may be able to do this.

4. sorting data, choosing the head or tail of data, grepping in or out the desired data

**p4subst**

The p4subst special operator is a stream editing operator, which is roughly analogous to doing

```
| sed 's/regular expression/literal/g' |
```

in a shell pipeline. The p4subst special operator should generally be used in an alias as:

```
something > $(output) &&
p4subst "regular expression" "literal replacement" < $(output) > $(result) &&
something else < $(result)
```

The regular expression in the p4subst special operator is interpreted using the same V8 Regexp library that
is used by 'p4 grep -e'.

## Practicalities

### Per-workspace aliases files
This technique, suggested by Jeff Anton, is pretty clever: in your P4CONFIG file, add the line:

```
P4ALIASES=$configdir/p4aliases.txt
```

Then, since your P4CONFIG file is found wherever you might be "underneath" it, but that location is
known by the special $configdir value, you can have a P4ALIASES file which is specific to this
workspace, and which is conveniently found no matter where you are in that workspace.

### Managing your aliases file

Right now, you simply use any text editor you want to edit your aliases file.

We're considering having a 'p4 alias <name> = <result>' command that would add-or-update an alias into
your aliases file without your needing to run an editor.

### Developing your alias

There are several long-form client-side command options which may be helpful:

**--aliases=dry-run!:** This expands the command through all the relevant alias processing, then shows you
the server command that it would have run, without actually running that server command.

**--aliases=echo!:** Similar to 'dry-run', but in addition to displaying the command that will be run, it also runs
it.
**--aliases=none!:** This option simply takes an immediate return from the aliases code. It doesn't even read
the aliases file, or care if you even have one. You can use this to run a command even if you have a syntax
error in your aliases file, and is also useful for running a command without using any aliases.

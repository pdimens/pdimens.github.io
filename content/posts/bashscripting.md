---
title: "Writing clean BASH scripts"
date: 2022-07-03T23:29:21+05:30
draft: false
author: "Pavel Dimens"
tags:
  - Tutorial
image: /images/bash/anchie_R.png
description: "Let's write bash scripts nicer and cleaner"
toc: 
---


While you don't need to know *everything* about `sh` or `bash` to write functional code, you'll find that like `R`, there are certain ways to write code that end up making life easier for you. This tutorial is meant to share some of those methods so you can have less headaches along the way. By no means is this a "you will be awesome at BASH after reading this", but rather a "this will make life easier." Remember to make your shell scripts executable with `chmod +x <script.name>` before you run them (otherwise the system doesn't know that it can execute it).



## Wrap it up

The things mentioned afterwards in the tutorial center around using "wrapper" scripts instead of executing things directly via command line. A wrapper (or shell) script is just an executable text file that contains the commands you want to run in the order you want to run them. So here's an example of a two direct commands that you might enter one after another:

```sh
cd /home/user/Music/
mkdir ./Madonna_Greatest_Hits
```

the wrapper script (lets call it `madonna.sh`) would be a text file that can look like:

```sh
#file: madonna.sh
#this script will create an empty folder for Madonna's Greatest Hits in your /Music folder

cd /home/user/Music/              # change directories
mkdir ./Madonna_Greatest_Hits     # make the empty folder
```

which you would execute with

```sh
./madonna.sh
```

So here are the immediate benefits to the wrapper script:

1. executing `madonna.sh` will automate all the commands you put within it
2. you can annotate inside the file, which means you don't need to remember every single little detail weeks, months, or years down the line.

**ok, but making empty folders to fill with Madonna's Greatest Hits isn't relevent to what you do**

So let's add to this concept with things that **are** important and relevant



## Add a shebang

In coding terminology, a "shebang" is a line at the top of a script that informs the computer what language interpreter it needs to use to correctly run your script. In other words, it tells the computer what language your code is in. All the coding languages on a computer are installed *somewhere*, so a shebang at the top of your script points to the language's location and cuts out the guesswork. If a system can't find the language, the script cannot be run. Computers by default don't "speak" all the coding languages (but common ones are usually installed by default), so you may need to occasionally install a language. The tricky thing is, different Unix-like operating systems (macOS, the different kinds of Linux) store those languages in different places due to personal preference. Conveniently, for `bash` or `sh` scripts, the "universal" shebang is `#!/usr/bin/env bash`, which should work on most/all systems. It should be similar for `ruby` (e.g. `#!/usr/bin/env ruby`) or python (e.g. `#!/usr/bin/env python`). So let's apply that to `madonna.sh`:

```sh
#!/usr/bin/env bash

# file: madonna.sh
# this script will create an empty folder for Madonna's Greatest Hits in your /Music folder

cd /home/user/Music/              # change directories
mkdir ./Madonna_Greatest_Hits     # make the empty folder
```

Now that we have that shebang there, we don't need to call the script `madonna.sh`, and if we wanted to, could rename it to just `make_madonna_folder` without an extension, since the script now indicates what language the computer need to use. These shebangs are how you can call up scripts from your `$PATH` that don't have filename extensions, like `dDocent`, `canu`, or `apt`.


## Incorporate variables

Fact is, you probably wont need to automate making a bunch of folders for great 80s music. The kinds of things we run, like `structure`, `pilon`, `vcftools`, require a bunch of arguments to tweak run parameters, along with input and output filenames. Often times, you will need to run and rerun these scripts to get them just right, each time tweaking some things here and there. The most time-saving way to do that is by using variables. For `bash` scripts, it's as simple as adding the variables between the shebang and the actual code with `VARIABLE=VALUE` (no spaces) then referencing it later in the code by adding a dollar sign in front of the variable. For example, if we were to set `SPICEGIRLS=1997`, then you would reference it with `$SPICEGIRLS`, which the script would interpret as `1997` since that's what you set the variable to.

Let's apply this idea to the actual code of an actual analysis. First, let's look at what a direct command line execution of a particular job looks like:

```sh
sh DBG2OLC k 17 AdaptiveTh 0.0001 KmerCovTh 2 MinOverlap 20 Contigs Contigs.txt f ~/assemblytest/Pacbio.fasta RemoveChimera 1
```

There's a lot of components to that, isn't there? There are several flags and arguments, directories for data, etc. It doesn't make for good legibility and changing bits will be kind of annoying. So, let's convert that into a shell script called `assemblytest.sh`:

```sh
#!/usr/bin/env bash

KMER=17                               # number of overlaps necessary
THRESH_VALUE=0.0001                   # error threshold
KCOV=2                                # convergence factor
OVERLAP=20                            # number of overlapping bases to look at
CONTIGS=Contigs.txt                   # input contig file
PACBIO=~/assemblytest/Pacbio.fasta    # input PacBio fasta file

~/DBG2OLC/DBG2OLC  k $KMER \
AdaptiveTh $THRESH_VALUE \
KmerCovTh $KCOV \
MinOverlap $OVERLAP \
Contigs $CONTIGS \
f $PACBIO \
RemoveChimera 1
```

**Let's dissect this script, because there are some goodies in here.**

1. You see the shebang at the top, so both we and the system know it's a `bash` script.
2. We have created a bunch of variables up at the top, each separated by a line and annotated for outselves.
3. Each flag (e.g. `AdaptiveTh` or `KmerCovTh`) is followed by referencing a variable that was set at the top. As an example, `MinOverlap $OVERLAP` is the same as `MinOverlap 20` because we set `OVERLAP=20` above.
4. Each of the arguments for the `DBG2OLC` script is separated by `\` and begins on a new line, which increases code legibility substantially. The `\` at the end of each line lets us start a new line without a break in the command.

**So instead of running that clunky single-line command, we would just run our shell script as:**

```sh
./assemblytest.sh
```

Remember, your script needs to be given executable permission to allow it to run, this can be done by the command

```sh
chmod +x <script.name>
```



## Add user inputs

If you're reading this, you've likely encountered scripts that require user inputs. If you haven't, the examples above demonstrate this very well. Inputs can take one of three forms:

1. User prompts, like `dDocent`
2. Flags and corresponding values after the command, like `gzip -d <file>` or `structure -m mainparams.dat -e extraparams.dat`
3. Values after a command in a particular order, like `punzip <working.director> <destination.dir> <keyword>`

If you're writing a script where you would like user input after the command ("arguments"), then we need to discuss some basics. 

### User prompts

Not used as often as some of the other methods and can be cumbersome for advanced users, but this is a totally valid way of adding arguments. The command is actually a simple one:

```sh
read VARIABLE
```

This will pause the script to allow a user to input some text that will be stored as the `VARIABLE` which can be called up later in the script with `$VARIABLE` as described above. Off the bat, the user gets not indication as to what input they need to add, so common practice is to use `echo` right before `read` to give the user an idea of what they are inputting. Here are some examples:

```sh
echo "What day of the week is it?"
read DAY_INPUT
echo "$DAY_INPUT is a great day to listen to ABBA"
```

```sh
# this is the tail end of the unpac script
echo "Do you want to process *scraps.bam files too? (y/n)"
read SCRAPS

if [ "$MODE" == 'multi' ] && [ "$SCRAPS" != 'y' ] ;
  then process_subreads
elif [ "$MODE" == 'multi' ] && [ "$SCRAPS" == 'y' ] ;
  then process_subreads &&  process_scraps
fi
```



### Flags as arguments

To use flags like in the `DBG2OLC` examples above, you will need to become acquainted with `getops`. This can be implemented in multiple ways, but the gist is that you specify what flags are possible with your script, whether they are a toggle (yes/no) or require a value, and what they do. By "toggle", we're referring to a flag that does not require an argument after it, rather, the flag itself is the argument. Perhaps an example would help:

```sh
gzip archive.tar		# this would compress archive.tar
```

versus 

```sh
gzip -d archive.tar.gz	# the -d flag indicates that gzip should decompress the file
```

Another example would be in the original version of `unpac`, where the `-s` flag, if present, indicated that the script should also process `.scraps.bam` sequence files.  

`getops` is a little tricky to figure out at first, and there are multiple ways if implementing it, so let's try to cover the basics that may get you started with it. First, you should know that there is `getopts` (short for "get options") and the older `getopt`. For the sake of covering superficial basics, you will need to know that `getopts` requires single-letter flags (e.g. `-n` `-f`, etc.), whereas `getopt` allows for double-dashed words (e.g. `--file`  `--dir`, etc.). While more verbose flags are preferred for code approachability, `getopts` are generally preferred, so let's cover that.



#### `getops` header

Including `getopts` into a bash script is as simple as adding a code chunk, in this case a `case` (described in a different tutorial) inside a `while` loop:

```sh
while getopts ":p:d:m:s" opt; do
  case $opt in
```

The important part of this loop is the stuff in the quotes `":p:d:m:s"`, where those letters are the flags you'll allow users to pass when running your script. In this example, the possible flags are `-p` `-d` `-m` `-s`. The colons are important, as the first flag `p` must have a colon to the left of it if your script requires flags at all (no colon on the left = flags are optional). After that, any flag that will require an argument requires a colon to the right of the letter. Looking at the example above, flags `p`, `d`, and `m` require arguments (have colons to the right of the letters), but `s` does not require an argument, therefore it has no colon to the right of it. You could just as well not require arguments from `m` or `d` but require them for `s`, which would look like `":p:dms:"`. And of course, you can change the letters to whatever suits your needs. This header is followed by the body of the loop.



#### `getopts` body

This is the real meat of your options. This section controls what happens when a flag is invoked and what an argument (if passed to it) gets used for. If you plan on being thorough, this section also includes things like error prompts if something is missing, incorrectly written, in the wrong format (e.g. numbers instead of letters), etc.  There are many different ways to implement the option handling, but the section follows the basic format of

```sh
    p)
      <stuff stuff stuff>
      ;;
    d)
      <stuff stuff stuff>
      ;;
    m)
      <stuff stuff stuff>
      ;;
    s)
      <stuff stuff stuff>
      ;;
	\?)
      echo "Invalid option: -$OPTARG" >&2
      usage  # we talk about "usage" in section 5 below
      exit 1
      ;;
    :)
      echo "-$OPTARG requires an argument." >&2
      usage
      exit 1
      ;;
  esac
done
```

The first thing you should notice is that the convention looks pretty similar to a lettered list, with the letters corresponding to the letters we provided in the header of the loop. Within each letter, you have put a series of commands or arguments, and end the section with two semicolons `;;`. There are two additional sections for when an unrecognized flag is used `\?)` and when a flag requiring an argument is invoked without an argument provided `:)`. 



##### `$OPTARG`

You now also suddenly have this `$OPTARG` variable. Simply put, any argument you pass to a flag becomes coded as `$OPTARG` within that flag's section in the body of this loop. I think the best way to clarify what that means is with an example. Here is an excerpt of actual code:

Let's say this is the command you put in:

```sh
./unpac.sh -p tomato -d ./eggplant/ -m multi
```

And this is what the getopts code looks like for `unpac.sh`

```sh
PREFIX=
WORKDIR=
MODE=

while getopts ":p:d:m:s" opt; do
  case $opt in
    p)
      echo "fasta file prefix set to: $OPTARG" >&2
      PREFIX=$OPTARG
      ;;
    d)
      WORKDIR=$OPTARG
      echo "working directory set to: $OPTARG"
      ;;
    m)
      MODE=$OPTARG
      echo "unpac will be run in $OPTARG mode"
      ;;
```

Based on how we ran the script, when `$OPTARG` appears within the `p)` section, it will refer to the argument you passed for the `-p` flag, which in this case is `tomato`.  So in that section, the script will now echo "fasta file prefix set to tomato" and set the variable `PREFIX` to `tomato`. Likewise, in the `d)` section, `$OPTARG` is `./eggplant/`, since that is the argument we passed to that flag, and in that section the `WORKDIR` variable is set to  `./eggplant/`, and the script will echo "working directory set to ./eggplant/". The same applies for the `-m` flag, which you should hopefully be able to deconstruct on your own.

Also do pay attention to the empty variables above the `getopts` section. If you plan on assigning any `$OPTARG` to variables, those variables must first be established outside of (and before) the loop. This can also sometimes be a way to set default values. 

If you are curious to see one kind of implementation of `getops`, refer to the `unpac` script found [here](https://github.com/pdimens/genomic-toolbox/blob/master/unpac/unpac).



### Values after commands

This is the simplest implementation of arguments. It requires less coding than flags, but the trade of is less flexibility in that arguments must be "passed" in a specific order (which is why it's good practice to have usage text appear when the script is run without arguments). To implement something like this, then you must understand argument passing. The command you enter into the command line is interpreted and recoded by the machine in a particular way, that is, the command itself is recoded as `$0`, and every argument after the base command increases the value after the `$` by 1. Let's look at an example of a *de novo* RAD reference assembly script from Dr. Jon Puritz included with `dDocent`:

```sh
remake_reference.sh 2 5 3 6 0.9 10
```

which is "recoded" by the machine as

```sh
$0 $1 $2 $3 $4 $5 $6
```

or more simply:

- `remake_reference.sh` is coded as `$0`
- the `2` after it is coded as `$1`
- the `5` after that is coded as `$2`
  - the `3` after that is coded as `$3 ` ...and so on...
- the final `10` is coded as `$6` 

Because of this, we can combine the idea of variables from Section 3 with this native recoding scheme to make up a simple script `list_things.sh` that outputs the list of files in a given directory to a file with a specific name:

```sh
#! /usr/bin/env bash

DIRECTORY=$1
OUTFILE=$2

cd $DIRECTORY
ls -l > $OUTFILE
```

or another way of writing it would also be 

```sh
#! /usr/bin/env bash

cd $1
ls -l > $2
```

but good practice is to have descriptive variables in your code, so the first example is recommended. This command would then be run:

```sh
list_things.sh ~/Music/Mr_Big/Lean_into_It/ album_songlist.txt
```

and it would parse my first argument, which is the directory to a specific Mr. Big album, perform `ls -l` there, and output that into `album_songlist.txt`



## Usage text

A great way to improve code legibility is the inclusion of what's known as "usage". This practice is very common for command-line applications, and its purpose is to be a short reminder/summary of what the code does and how to use it. Usages are typically invoked by calling a script without any arguments, as so:

```sh
>unpac

This script will convert PacBio bam files into fasta format using samtools,
then it concatenates them into a single file and counts the number of base
pairs and total number of contigs in the concatenated file.
        [Usage]
unpac -p <output.prefix> -d <working.directory> -m multi -s
-p <prefix>    | prefix output for renamed final fasta files
-d <directory> | path to working directory
-m <mode>      | "single" or "multi" mode (without quotes)
-s             | toggle to also process scraps.bam (kind of time consuming)
```

I wrote this code and use it fairly often, but even I need reminders of how to use it right, so if nothing else, usages are helpful to you too. There are two things that need to be in your script to activate usage text when no arguments/flags are passed:

1. The usage text, preferably wrapped in a function (legibility and convenience)
2. An `if` conditional recognizing when no arguments are passed

This is actually pretty easy to do(!), so let's take care of the text first.

### Functions

All you really need here is a function called `usage` that invokes the `cat <<EOF` command and has your text within it. It follows this syntax:

```sh
usage() {
cat <<EOF
This script will convert PacBio bam files into fasta format using samtools,
concatenating them into a single file and counting the number of base pairs 
and total number of contigs in the concatenated file.
        [Usage]
$0 -p <output.prefix> -d <working.directory> -m multi -s
-p <prefix>    | prefix output for renamed final fasta files
-d <directory> | path to working directory
-m <mode>      | "single" or "multi" mode (without quotes)
-s             | toggle to also process scraps.bam (kind of time consuming)
EOF
}
```

The function must end with `EOF` before the closing curly brace. What content you put in there is up to you. Common convention is to have a brief explanation, an example of usage, and an explanation of the possible flags, although not necessarily in the format above (it is *your* code after all!).



### `if` conditionals

All you need here is a simple chunk of code that says "if no arguments are given, run `usage`", which looks exactly like this:

```sh
if [[ -z "$1" ]]; then
  usage
  exit 1
fi
```

The `-z` means "equals zero", therefore `[[ -z "$1"]]` means "if the first argument equals zero (meaning there's nothing there), then" and we just tell it to run `usage` and exit out of the script. Easy peasy!		
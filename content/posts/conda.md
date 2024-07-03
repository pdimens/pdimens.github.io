---
title: "Using Conda in your workflow"
date: 2021-04-03T23:29:21+05:30
draft: false
author: "Pavel Dimens"
tags:
  - Tutorial
image: /images/conda/conda.jpg
description: "Why conda is the best... ever"
toc: 
---

At some point, you may have come across tutorials and posts online referring to `conda` `Anaconda` or `miniconda` and just ignored it (we've all been there, no judgement). Hopefully by the end of this writing, you will be convinced that `conda` can be a big deal and make your life quite a bit easier. 

## Virtual environments
Why bother?

If you've read a tutorial for the command `screen` and have used it, or run something inside a virtualbox, you may already understand why virtual environments can be great. Conceptually, a virtual environment creates a little isolated pocket on your system to run things and not really interact with things outside of it (generally). When you use a virtualbox, you're using an operating system inside of your native operating system, without the two really clashing. When you're using `screen`, you create virtual terminal sessions that are independent of your main session that you can attach/detach at will and prevent jobs terminating on exiting your main session. So, what if you need to compile source code or run a particular piece of software that requires a different version of something than is already on your system? Say, for example you're trying to compile `blasr` and its various dependencies, some of whom have cmake config files that require python2, others that require python3 (some commands from python2 are deprecated in python3... the exact ones you need for this particular installation), but all of them just point to "python" and the build fails and fails and fails until you give up. 

## Welcome to Conda

`conda` is really kind of a simple concept. You create a new conda environment by invoking `conda` and giving it a name, then install software into this environment. 

To use any of the things you install into that environment, you just enter that environment (with a command), and invoke the commands as if they're already in your `$PATH` (but they aren't!). If you're outside of this conda environment, you cannot invoke these programs because as far as your system is concerned, they were never really installed onto the system. Even better, you can likely find what you need through `bioconda` or `conda-forge`, super convenient "channels" that have **many** bioinformatics programs ready available for simple installation without the insanity of compiling. On a linux system, the `conda` stuff (environments, software you install, etc.) all live in a simple folder at `home/<user>/.conda` and you can browse through it without any fuss.

## Using conda

Let's use a real-world example of installing the genome assembler `DBG2OLC`, whose dependencies are things like `blasr` `hstlib` `hdf5` `sparc` `sparseassembler` and what seems like a million other things. The real monster of the bunch is `blasr` which has a dated installer (thanks, PacBio) using `python2` and wont compile because the configuration parameters use calls that are deprecated in python3 (which pretty much every system has installed and is used by default with some exceptions). The short of it is, without **a lot** of intervention, this will absolutely not compile, and you won't assemble genomes. **But**, all of those things are readily available through bioconda, so let's install it through that. When in doubt, check the [Anaconda Cloud](https://anaconda.org/) if the software you're looking for has an installation recipe!

**Let's create a conda environment called `assemblers`**

```sh
conda create --name assemblers
```

- `create` creates the environment
- `--name` (or `-n`) gives that environment a name, which for us is `assemblers`

**Then we activate the environment and begin installing stuff**

```sh
conda activate assemblers
conda install -c bioconda blasr
```

- `-c` calls up a "channel", which for us is `bioconda`
- the last part, `blasr` is the software we want to install (from bioconda) into the environment

Then you'll install the rest of the stuff you need in there

```bash
conda install -c bioconda sparc sparseassembler dbg2olc abyss pbdagcon
```

Then, just call up what you need (while in your environment) as though it's been installed on your system proper, and that's it!

## Using conda on an HPC

The HPC works a little differently than our workstations. Mainly, instead of inputting a command, pressing `enter` and the job running, the HPC takes "job scripts" and decides when and how to run them depending on the amount of resources other users are using at that time. For the most part, the only change in your basic computer work would be the addition of a header to your scripts that the scheduler parses, but if you are relying on `conda` environments, then you need to do a little extra.  Your script will need to include (depending on the version of `conda` you are using):

```bash sh
source activate <conda_env>  #anaconda2
```

or 

```bash
conda activate <conda env>  # anaconda3
```

before any other commands in your job file.
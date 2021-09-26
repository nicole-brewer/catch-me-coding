---
toc: true
layout: post
description: A brief introduction to submitting jobs with SLURM for those with basic experience on using an HPC cluster.
categories: [markdown]
title: Simple SLURM Tutorial
mathjax: true
---

## Submitting Jobs with SLURM

**srun** is used to submit a job for execution or initiate job steps in real time. srun
has a wide variety of options to specify resource requirements

### Example

The command `hostname` ouputs the hostname and domain of the system it is run on in the following format: `hostname.domain`.

Try it for yourself:

```bash
$ hostname
```

Now we can try submitting a SLURM job that can run this command on our available resources in various ways.

First, execute /bin/hostname in six tasks with the *-n6* option:

```bash
$ srun -n6 -l hostname 
```

*Note:  the `-l` option prepends the lines of output with the task number*

Next, execute `/bin/hostname` on three separate nodes with the *-N3* option:

```bash
$ srun -N3 -l hostname 
```

Now try both options together and notice that the default behavior is to divy up the tasks equally among the nodes.

These are just two of the *many* different ways to specify resource requirements including: minimum
and maximum node count, processor count, and specific nodes to use or not use. We will learn more about these options as we go along, but a complete list can be found [here](https://slurm.schedmd.com/srun.html).

Now we aren't on this cluster all by ourselves, so typically we want to submit a job or a series of jobs to the queue to wait it's turn in line to be executed. **sbatch** is used to submit a job script for later execution. The script will typically contain one or more **srun** commands to launch parallel tasks.

For example, we can submit our last exaple job to the queue with the following script:

```bash
#!/bin/bash
#SBATCH -J first_example  # Job Name
#SBATCH --time=00:00:30   # Walltime
#SBATCH -o output.txt     # The destination file for stdout
#SBATCH -e error.txt      # The destination file for stderr

srun hostname
srun echo "whatever"
```

This naming schema for output and errors isn't always ideal, because subsequent submission of this script will overwrite these files. To prevent this, we can use **replacement symbols** to customize the filename patterns. Here are some common ones:  

**%j** - jobid of the running job. 

**%n** - Node identifier relative to current job (e.g. "0" is the first node of the running job) This will create a separate IO file per node.  

**%t** -  task identifier (rank) relative to current job. This will create a separate IO file per task.  

**%u** - User name.  

**%x** - Job name. 

For example we can change the output and error files to include the jobID unique to each submission.

```bash
#SBATCH -o jobID_%j.o     # The destination file for stdout
#SBATCH -e jobID_%j.e     # The destination file for stderr
```

This way we can remove these files relating to one job in particular very easily. Say the jobID of our last job was 292. We can easily remove the files for this run (but not others) by calling:

```bash
$ rm jobID_292*
```

Or if we want to list the files in numeric order we can use the command:

```bash
$ ls | sort -n -t _ -k 2
```

The `sort` command has lot of options that makes manipulating token separated file schemas very easily. The `-n` flag indicates the sort should be strictly numeric. The `-t _` option specifies that sections are separated by underscores, and the `-k 2` option indicates that the second "chunk" in between the underscores is the one we want to sort by.

If you are unfamiliar with the pipe symbol (`|`), refer to [this](https://ryanstutorials.net/linuxtutorial/piping.php) tutorial. 



## Performance Tuning

In this section, we will learn about how our cluster architecture influences our resource requirements and runtime performance.

### Terminology

- A **node** contains one or more sockets. 4

- A **socket** is a receptacle on the motherboard for exactly one processor. 2

- A **processor** contains one or more (CPU) cores. 20

- For our purposes, a one **task** is usually assigned to each CPU. (If not the `-c` option can be used to specify cores per task.





![Figure: Definitions of Socket, Core, & Thread](https://slurm.schedmd.com/mc_support.gif)



We want to know about Haynes specifically. SLURM has the following command that provides us with useful information about the hardware on our nodes.

```bash
$ scontrol show node
```

First, you will notice that we have five nodes. Next, take note of `CoresPerSocket, Sockets, Boards`and `ThreadsPerCore`.

```bash
CoresPerSocket=20
Sockets=2
Boards=1
ThreadsPerCore=1
```

We can verify that `cpu=40` by working out the fractions, $\frac{1 \text{ motherboard}}{\text{Node}}*\frac{2 \text{ sockets}}{\text{motherboard}}*\frac{ 20\text{ cores}}{\text{socket}}= \frac{40 \text{ cores}}{\text{node}}$ .



***Running

tpv33_receivers.dat 'tpv33_initial_stress.yaml' tpv33_faultreceivers.dat' 'output/data'

___



**%A** - Job array's master job allocation number.  

**%a** - Job array ID (index) number.  

**%J** - jobid.stepid of the running job. (e.g. "128.0")  

**%N** - short hostname. This will create a separate IO file per node.  

**%s** - stepid of the running job.  

```bash
#!/bin/bash
#SBATCH -J array_job
#SBATCH -o jobID_%A_taskID_%a.out
#SBATCH -e jobID_%A_taskID_%a.err
#SBATCH -t 00:05:00
#SBATCH --array=1-20
#SBATCH -n 1

echo "SLURM_ARRAY_TASK_ID is $SLURM_ARRAY_TASK_ID" >> result_$SLURM_ARRAY_JOB_ID.out
echo "Waiting $SLURM_ARRAY_TASK_ID seconds..."
echo "All done!"
```








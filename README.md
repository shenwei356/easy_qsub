# easy_qsub

Easily submitting multiple PBS jobs or running local jobs in parallel. Multiple input files supported.

## Submitting PBS jobs

easy_qsub submits PBS jobs with script template, avoid repeatedly editing PBS scripts.

Default template (```~/.easy_qsub/default.pbs```):

```
#PBS -S /bin/bash
#PBS -N $name
#PBS -q $queue
#PBS -l ncpus=$ncpus
#PBS -l mem=$mem
#PBS -l walltime=$walltime
#PBS -V

cd $$PBS_O_WORKDIR
echo run on node: $$HOSTNAME >&2

$cmd
```
Generated PBS scripts are saved in ```/tmp/easy_qsub-user```.
If jobs are submitted successfuly, PBS scripts will be moved to current directory.
If not, they will be removed.

## Support for multiple inputs

Inspired by [qtask](https://github.com/mbreese/qtask), **multiple inputs**  is supported
 (See example 2). If "{}" appears in a command, it will be replaced
with the current filename. Four formats are supported.
For example, for a file named "/path/reads.fq.gz":

<table>
    <tr>
        <th>format</th>
        <th>target</th>
        <th>result</th>
    </tr>
    <tr>
        <td>{}</td>
        <td>full path</td>
        <td>/path/reads.fq.gz</td>
    </tr>
    <tr>
        <td>{%}</td>
        <td>basename</td>
        <td>reads.fq.gz</td>
    </tr>
    <tr>
        <td>{^.fq.gz}</td>
        <td>remove suffix from full path</td>
        <td>/path/reads</td>
    </tr>
    <tr>
        <td>{%^.fq.gz}</td>
        <td>remove suffix from basename</td>
        <td>reads</td>
    </tr>
</table>


## Running local jobs in parallel

It also support runing commands locally with option ```-lp``` (parallelly) or ```-ls``` (serially).
This make it **easy to switch between cluster and local machine**.


## Best partner: ```cluster_files```

To make best use of the support for multiple input, a script ```cluster_files``` is added to
cluster files into multiple directories by creating symbolic links or moving files (See example 3,4).
**It's useful for programs which take one directory as input.**

Another useful scene is to apply different jobs to a same dataset. One bad directory structure is:

    datasets/
    ├── A
    ├── A.stage1
    ├── A.stage2
    ├── B
    ├── B.stage1
    └── B.stage2

A flexible structure can be organsize by `cluster_files`.
Instead of changing original directory structure, using links could be more clear and flexible.

    datasets
    ├── A
    └── B
    datasets.stage1
    ├── A
    └── B
    datasets.stage2
    ├── A
    └── B



## Examples

1) Submit a single job

    easy_qsub 'ls -lh'

2) Submit multiple jobs, runing fastqc for a lot of fq.gz files

    easy_qsub -n 8 -m 2GB 'mkdir -p QC/{%^.fq.gz}.fastqc; zcat {} | fastqc -o QC/{%^.fq.gz}.fastqc stdin' *.fq.gz

Excuted commands are:

	mkdir -p QC/read_1.fastqc; zcat read_1.fq.gz | fastqc -o QC/read_1.fastqc stdin
	mkdir -p QC/read_2.fastqc; zcat read_2.fq.gz | fastqc -o QC/read_2.fastqc stdin

Dry run with -vv

    easy_qsub -n 8 -m 2GB 'mkdir -p QC/{%^.fq.gz}.fastqc; zcat {} | fastqc -o QC/{%^.fq.gz}.fastqc stdin' *.fq.gz -vv

3) Supposing a directory ```rawdata``` containing **paired files** as below.

    $ tree rawdata
    rawdata
    ├── A2_1.fq.gz
    ├── A2_1.unpaired.fq.gz
    ├── A2_2.fq.gz
    ├── A2_2.unpaired.fq.gz
    ├── A3_1.fq.gz
    ├── A3_1.unpaired.fq.gz
    ├── A3_2.fq.gz
    ├── A3_2.unpaired.fq.gz
    └── README.md


And I have a program ```script.py```, which takes a directory as input and do some thing
with the **paired files**. Command is like this, ```script.py dirA```.

It is slow by submiting jobs like example 2), handing A2_\*.fq.gz and then A3_\*.fq.gz.
We can split ```rawdata``` directory into multiple directories (cluster files by the prefix),
and submit jobs for all directories.

	cluster_files -p '(.+?)_\d\.fq\.gz$' rawdata -o rawdata.cluster

    tree rawdata.cluster/
    rawdata.cluster/
    ├── A2
    │   ├── A2_1.fq.gz -> ../../rawdata/A2_1.fq.gz
    │   └── A2_2.fq.gz -> ../../rawdata/A2_2.fq.gz
    └── A3
        ├── A3_1.fq.gz -> ../../rawdata/A3_1.fq.gz
        └── A3_2.fq.gz -> ../../rawdata/A3_2.fq.gz

	easy_qsub 'script.py {}' rawdata.split/*

Another example (e.g. some assembler can handle unpaired reads too):

    cluster_files -p '(.+?)_\d.*\.fq\.gz$' rawdata -o rawdata.cluster2

	tree rawdata.cluster2
    rawdata.cluster2
    ├── A2
    │   ├── A2_1.fq.gz -> ../../rawdata/A2_1.fq.gz
    │   ├── A2_1.unpaired.fq.gz -> ../../rawdata/A2_1.unpaired.fq.gz
    │   ├── A2_2.fq.gz -> ../../rawdata/A2_2.fq.gz
    │   └── A2_2.unpaired.fq.gz -> ../../rawdata/A2_2.unpaired.fq.gz
    └── A3
        ├── A3_1.fq.gz -> ../../rawdata/A3_1.fq.gz
        ├── A3_1.unpaired.fq.gz -> ../../rawdata/A3_1.unpaired.fq.gz
        ├── A3_2.fq.gz -> ../../rawdata/A3_2.fq.gz
        └── A3_2.unpaired.fq.gz -> ../../rawdata/A3_2.unpaired.fq.gz


4) Another example (complexed directory structure)

    tree rawdata2
    rawdata2
    ├── OtherDir
    │   └── abc.fq.gz.txt
    ├── S1
    │   ├── A2_1.fq.gz
    │   ├── A2_1.unpaired.fq.gz
    │   ├── A2_2.fq.gz
    │   ├── A2_2.unpaired.fq.gz
    │   ├── A4_1.fq.gz
    │   └── A4_2.fq.gz
    └── S2
        ├── A3_1.fq.gz
        ├── A3_1.unpaired.fq.gz
        ├── A3_2.fq.gz
        └── A3_2.unpaired.fq.gz

    cluster_files -p '(.+?)_\d\.fq\.gz$' rawdata2/

    tree rawdata2.cluster/
    rawdata2.cluster/
    ├── A2
    │   ├── A2_1.fq.gz -> ../../rawdata2/S1/A2_1.fq.gz
    │   └── A2_2.fq.gz -> ../../rawdata2/S1/A2_2.fq.gz
    ├── A3
    │   ├── A3_1.fq.gz -> ../../rawdata2/S2/A3_1.fq.gz
    │   └── A3_2.fq.gz -> ../../rawdata2/S2/A3_2.fq.gz
    └── A4
        ├── A4_1.fq.gz -> ../../rawdata2/S1/A4_1.fq.gz
        └── A4_2.fq.gz -> ../../rawdata2/S1/A4_2.fq.gz

    cluster_files -p '(.+?)_\d\.fq\.gz$'  rawdata2/ -k -f  # keep original dir structure 

    tree rawdata2.cluster/
    rawdata2.cluster/
    ├── S1
    │   ├── A2
    │   │   ├── A2_1.fq.gz -> ../../../rawdata2/S1/A2_1.fq.gz
    │   │   └── A2_2.fq.gz -> ../../../rawdata2/S1/A2_2.fq.gz
    │   └── A4
    │       ├── A4_1.fq.gz -> ../../../rawdata2/S1/A4_1.fq.gz
    │       └── A4_2.fq.gz -> ../../../rawdata2/S1/A4_2.fq.gz
    └── S2
        └── A3
            ├── A3_1.fq.gz -> ../../../rawdata2/S2/A3_1.fq.gz
            └── A3_2.fq.gz -> ../../../rawdata2/S2/A3_2.fq.gz


## Installation

`easy_qsub` and `cluster_files` is a single script written in Python using standard library.
It's Python 2/3 compatible, version 2.7 or later.

You can simply save the script [easy_qsub](https://raw.githubusercontent.com/shenwei356/easy_qsub/master/easy_qsub)
and [cluster_files](https://raw.githubusercontent.com/shenwei356/easy_qsub/master/cluster_files)
to directory included in environment PATH, e.g ```/usr/local/bin```.

Or

    git clone https://github.com/shenwei356/easy_qsub.git
    cd easy_qsub
    sudo copy easy_qsub cluster_files /usr/local/bin

## Usage

easy_qsub

```
usage: easy_qsub [-h] [-lp | -ls] [-N NAME] [-n NCPUS] [-m MEM] [-q QUEUE]
                 [-w WALLTIME] [-t TEMPLATE] [-o OUTFILE] [-v]
                 command [files [files ...]]

Easily submitting PBS jobs with script template. Multiple input files
supported.

positional arguments:
  command               command to submit
  files                 input files

optional arguments:
  -h, --help            show this help message and exit
  -lp, --local_p        run commands locally, parallelly
  -ls, --local_s        run commands locally, serially
  -N NAME, --name NAME  job name
  -n NCPUS, --ncpus NCPUS
                        cpu number [logical cpu number]
  -m MEM, --mem MEM     memory [5gb]
  -q QUEUE, --queue QUEUE
                        queue [batch]
  -w WALLTIME, --walltime WALLTIME
                        walltime [30:00:00:00]
  -t TEMPLATE, --template TEMPLATE
                        script template
  -o OUTFILE, --outfile OUTFILE
                        output script
  -v, --verbose         verbosely print information. -vv for just printing
                        command not creating scripts and submitting jobs

Note: if "{}" appears in a command, it will be replaced with the current
filename. More format supported: "{%}" for basename, "{^suffix}" for clipping
"suffix", "{%^suffix}" for clipping suffix from basename. See more:
https://github.com/shenwei356/easy_qsub
```

cluster_files

```
usage: cluster_files [-h] [-o OUTDIR] [-p PATTERN] [-k] [-m] [-f] indir

clustering files by regular expression [V3.0]

positional arguments:
  indir                 source directory

optional arguments:
  -h, --help            show this help message and exit
  -o OUTDIR, --outdir OUTDIR
                        out directory [<indir>.cluster]
  -p PATTERN, --pattern PATTERN
                        pattern (regular expression) of files in indir. if not
                        given, it will be the longest common substring of the
                        files. GROUP (parenthese) should be in the regular
                        expression. Captured group will be the cluster name.
                        e.g. "(.+?)_\d\.fq\.gz"
  -k, --keep            keep original dir structure
  -m, --mv              moving files instead of creating symbolic links
  -f, --force           force file overwriting, i.e. deleting existed out
                        directory

```

## Copyright

Copyright (c) 2015-2017, Wei Shen (shenwei356@gmail.com)

[MIT License](https://github.com/shenwei356/easy_qsub/blob/master/LICENSE)

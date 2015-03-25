# easy_qsub

Easily submiting PBS job with script template, avoid repeatedly editing PBS scripts.

Inspired by [qtask](https://github.com/mbreese/qtask), support for multiple input
files is added (**See example 2)**.). If "{}" appears in a command, it will be replaced
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

**[Update]** To making best use of the support for multiple input, a script ```splitdir``` is added to
split directory into multiple directories by creating symbolic links or moving files.
**It's useful for programs which take directory as input*.**

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
echo run: $cmd >&2

$cmd
```

Generated scripts are saved in ```/tmp/easy_qsub-user```.

## Installation

easy_qsub is a single script written in Python using standard library. 
It's Python 2/3 compatible.

You can simply save the [script](https://raw.githubusercontent.com/shenwei356/easy_qsub/master/easy_qsub)
to directory included in environment PATH, e.g ```/usr/local/bin```.

Or
    
    git clone https://github.com/shenwei356/easy_qsub.git
    copy easy_qsub /usr/local/bin
    
## Usage

easy_qsub

```
usage: easy_qsub [-h] [-N NAME] [-n NCPUS] [-m MEM] [-q QUEUE] [-w WALLTIME]
                 [-t TEMPLATE] [-o OUTFILE] [-v]
                 command [files [files ...]]

Easily submit PBS jobs with script template. Multiple input files supported.

positional arguments:
  command               command to submit
  files                 input files

optional arguments:
  -h, --help            show this help message and exit
  -N NAME, --name NAME  job name
  -n NCPUS, --ncpus NCPUS
                        cpu number
  -m MEM, --mem MEM     memory
  -q QUEUE, --queue QUEUE
                        queue
  -w WALLTIME, --walltime WALLTIME
                        walltime
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

splitdir

```
usage: splitdir [-h] [-t TAG] [-s SUFFIX] [-m] [-f] indir

Split directory

positional arguments:
  indir                 source directory

optional arguments:
  -h, --help            show this help message and exit
  -t TAG, --tag TAG     output directory tag
  -s SUFFIX, --suffix SUFFIX
                        files/dirs common suffix (regular expression). if not
                        given, it will be the longest common substring of the
                        files
  -m, --mv              moving files instead of creating symbolic links
  -f, --force           force file overwriting, i.e. deleting existed out
                        directory

```


## Examples
    
1) Submit a single job

    easy_qsub 'ls -lh'

2) Submit multiple jobs, runing fastqc for a lot of fq.gz files

    easy_qsub -n 8 -m 2GB 'mkdir -p QC/{%^.fq.gz}.fastqc; zcat {} | fastqc -o QC/{%^.fq.gz}.fastqc stdin' reads/*.fq.gz

3) Supposing a directory ```rawdata``` containing *paired files* as below. 

	rawdata/
	├── A2_1.fq.gz
	├── A2_2.fq.gz
	├── A3_1.fq.gz
	└── A3_2.fq.gz

And I have a program ```script.py```, which takes a directory as input and do some thing
with the *paired files*. Command is like this, ```script.py dirA```.

It is slow by submiting jobs like example 2), handing A2_*.fq.gz 
and then A3_*.fq.gz. So we can split ```rawdata``` directory into multiple directories, and
submit jobs for all directories.

	splitdir -t 'sub' -s '_\d.fq.gz' rawdata/
	
	$ tree
	.                                                                                                      
	├── rawdata                                                                                            
	│   ├── A2_1.fq.gz                                                                                     
	│   ├── A2_2.fq.gz
	│   ├── A3_1.fq.gz
	│   └── A3_2.fq.gz
	├── rawdata.sub.A2
	│   ├── A2_1.fq.gz -> ../rawdata/A2_1.fq.gz
	│   └── A2_2.fq.gz -> ../rawdata/A2_2.fq.gz
	└── rawdata.sub.A3
		├── A3_1.fq.gz -> ../rawdata/A3_1.fq.gz
		└── A3_2.fq.gz -> ../rawdata/A3_2.fq.gz
	
	easy_qsub 'script.py {}' rawdata.sub.*

## Copyright

Copyright (c) 2015, Wei Shen (shenwei356@gmail.com)

[MIT License](https://github.com/shenwei356/easy_qsub/blob/master/LICENSE)
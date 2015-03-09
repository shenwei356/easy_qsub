# easy_qsub

Easily submit PBS job with script template, avoid repeatedly editing PBS scripts.

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

Generated scripts are saved in ```/tmp/easy_qsub```.

## Installation

easy_qsub is a single script written in Python using standard library. 
It's Python 2/3 compatible.

You can simply save the [script](https://raw.githubusercontent.com/shenwei356/easy_qsub/master/easy_qsub)
to directory included in environment PATH, e.g ```/usr/local/bin```.

Or
    
    git clone https://github.com/shenwei356/easy_qsub.git
    copy easy_qsub /usr/local/bin
    
## Usage

```
usage: easy_qsub [-h] [-v] [-N NAME] [-n NCPUS] [-m MEM] [-q QUEUE]
                 [-w WALLTIME] [-t TEMPLATE] [-o OUTFILE]
                 cmd [fileglob]

Easily submit job to pbs with template

positional arguments:
  cmd                   command to submit
  fileglob              file glob expression

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         verbosely print information
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
```

## Examples
    
1) Submit a single job

    easy_qsub 'ls -lh'

2) Submit multiple jobs

	easy_qsub -n 8 -m 2GB 'mkdir -p QC/{%^.fq.gz}.fastqc; zcat {} | fastqc -o QC/{%^.fq.gz}.fastqc stdin' reads/\*.fq.gz


## Copyright

Copyright (c) 2015, Wei Shen (shenwei356@gmail.com)

[MIT License](https://github.com/shenwei356/easy_qsub/blob/master/LICENSE)
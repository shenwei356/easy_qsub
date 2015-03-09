# easy_qsub


Easily submit job to pbs with template

## Usage
   
    usage: easy_qsub [-h] [-v] [-N NAME] [-n NCPUS] [-m MEM] [-q QUEUE]
                     [-w WALLTIME] [-t TEMPLATE] [-o OUTFILE]
                     cmd
    
    qsub with template
    
    positional arguments:
      cmd                   command to submit
    
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
                            template script
      -o OUTFILE, --outfile OUTFILE
                            output script

## Examples
    
1) submit a single job

    easy_qsub 'ls -lh'

2) submit multiple jobs

	todo


## Copyright

Copyright (c) 2015, Wei Shen (shenwei356@gmail.com)

[MIT License](https://github.com/shenwei356/easy_qsub/blob/master/LICENSE)
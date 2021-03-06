Useful bash aliases
===================

Snakemake is very flexible in workflow execution, see `its documentation <https://snakemake.readthedocs.io/en/stable/executable.html#all-options>`_ for detailed description.
Here you may find some useful aliases for multi-threaded execution, running on SGE cluster or visualization of individual steps of the pipeline.

* dsnake - shows rules to be executed, but do not start analysis (--dryrun)
* vsnake - visually examine rules to be executed and their dependencies
* snake  - runs standard SnakeLines pipeline in the current working directory
* qsnake - distribute snake jobs on the cluster - use for multi-threaded rules
* fsnake - distribute snake jobs on the cluster - use for single-threaded rules

Running SnakeLines
------------------

Edit your ~/.bash_aliases file, to load aliases at login to the system.

.. code-block:: bash

   vim ~/.bash_aliases

Add aliases for using pipelines locally

.. code-block:: bash

   # Basic
   THREADS_LOCAL=4     # Number of used threads, when running pipelines locally
   SNAKELINES_DIR=/data/snakelines/$USER   # Path to snakelines source files

   alias basesnake='snakemake -d `pwd` --jobname {rulename}.{jobid} --reason --printshellcmds --config snakelines_dir=$SNAKELINES_DIR'
   alias snake='basesnake threads=$THREADS_LOCAL --cores $THREADS_LOCAL'
   alias dsnake='basesnake threads=$THREADS_LOCAL --dryrun'

   function vsnake {
      dsnake $@ --rulegraph | dot | display
   }

You may also run scripts on computational cluster

.. code-block:: bash

   # SGE job scheduling system specific
   SGE_PRIORITY=0   # Priority of tasks on cluster
   SGE_THREADS=16   # Number of used threads, when running pipelines on cluster
   SGE_NODES=10     # Number of computational nodes in the cluster
   SGE_CORES=160    # Total number of cpus on all nodes in the cluster
   SGE_PARAMS="qsub -cwd -o log/ -e log/ -p $PRIORITY -r yes" # Additional parameters for the SGE scheduler

   function clustersnake {
      NOW=`date "+%y-%m-%d_%H-%M-%S"`
      LOGDIR=log/$NOW
      mkdir -p $LOGDIR
      rm -f log/last
      ln -f -s $NOW log/last
      basesnake threads=$1 --cluster "$CLUSTER_PARAMS -l thr=$1 -o $LOGDIR -e $LOGDIR" --jobs $2 ${@:3}
   }

   alias qsnake='clustersnake $SGE_THREADS $SGE_NODES'
   alias fsnake='clustersnake 1 $SGE_CORES'

Next time, you will log into system, aliases would be ready.
When you do not want to re-login, you may reload aliases

.. code-block:: bash

   source ~/.bash_aliases

Logging
-------

Logs are stored in the logs/ directory, when running pipeline on the computational cluster using qsnake or fsnake alias.
Each pipeline run has its own directory named by time of the execution.
The last/ directory is link to the last executed analysis.
For example:
::

   logs/
      |-- 18-10-08_16-19-48/
      |-- 18-10-08_17-16-23/
      |-- last -> 18-10-08_17-16-23/

Summary logs of the last run may be examined using log alias.
You must be inside of project directory to find which logs to display.

.. code-block:: bash

   function log {

      TYPE=e
      if [ "$1" == "o" ]; then
         TYPE=o
      elif [ "$1" == "a" ]; then
         TYPE=
      fi

      cat `python -c 'import os; cwd = os.getcwd(); print("/".join(cwd.split("/")[:4]))'`/log/last/*.$TYPE* | less
   }

Command ``log e`` will display only error messages, ``log o`` messages on standard stream.

Example run
-----------

Assuming you have input file in the SnakeLines compatible project structure, you may start analysis using these aliases

.. code-block:: bash

   # Go to screen - analysis would not terminate, if connection fails
   screen

   # Run always in the project root directory
   cd /data/projects/example

   # First try dryrun, check if pipeline is correct
   dsnake -s scripts/snake/process.snake process

   # Optionally visualise pipeline - but only on rules with small number of samples
   vsnake -s scripts/snake/process.snake test_process

   # Run test analysis with one, small sample
   snake -s scripts/snake/process.snake test_process

   # Distribute tasks for all samples on cluster
   ## For multi-threaded analysis
   qsnake -s scripts/snake/process.snake process
   ## For single-threaded analysis
   fsnake -s scripts/snake/process.snake process

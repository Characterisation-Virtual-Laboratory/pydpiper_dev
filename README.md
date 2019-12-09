Building the container
======================

The file pydpiper.def is a singularity build definition. You should be able to build it using the singularity build command

Running the Container
=====================

Use `singularity shell <container path>`

Set some environment variables

```
. /opt/torch/install/bin/torch-activate
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/minc/1.9.17/lib/
export PATH=$PATH:/opt/minc/1.9.17/bin
export PERLLIB=/opt/minc/1.9.17/perl
```

Ideally these environment variables should be set in the def file, but I haven't got around to that yet

Running the test data
---------------------

The authors provide a test dataset and a script `test_MBM_and_MAGeT.py`.

The first thing to do is edit this script to set the variable `num_execs` and the argument to `--mem`. I suggest setting `num_execs` to 2 and `--mem=10`. Note that this will require a 2 core, 20GB RAM computer or job allocation

When executing with larger values for `num_execs` I found the pipeline 'stalled' frequently. That is: the CPU usage dropped off and no more completed stages were written to the completed stages log.  It is worthwhile watching the completed log and making sure stages are still being written every few minutes at least.

Running the test data on M3
---------------------------

A precompiled singularity container is avilalbe on m3 as `/usr/local/pydpiper/pydpiper.sif`

Pydpiper has large storage requirements. The 1.1GB test dataset produced over 60GB of output data. I believe it needed closer to 100GB during execution.

Ideally execution on M3 will invove running the MBM.py process in one job and using qbatch to submit other jobs, but that requires extra work (Setting up qbatch). In the meantime, use the command smux

```
smux new-session --ntasks 2 --mem 25GB --time 2-0:0:0
```

To request 2CPUs and 25GB ram for 2 days.
You will the receive a standard prompt where you can use `singularity shell` as above.

smux will allow you to close your computer and persist the connection.  See information at https://docs.massive.org.au/M3/slurm/interactive-jobs.html. Note that smux is a wrapper around the standard utility tmux which allows you to create new shell windows split windows etc. See google for more information.

Running your own data
---------------------

In theory the command you want to run looks something like 
```
MBM.py
  --pipeline-name={MBM_name}
  --num-executors=2
  --verbose
  --init-model={datadir}/Pydpiper-init-model-basket-may-2014/basket_mouse_brain.mnc
  --lsq6-large-rotations
  --no-run-maget
  --maget-no-mask
  --mem=10
  --lsq12-protocol={datadir}/default_linear_MAGeT_prot.csv
  --local
  --files {datadir}/test-images/*.mnc
```

Where you replace  `{MBM_name}` with an identifier for your dataset, and `{datadir}` with the path to your data. I'm not sure what you should set the init-model to.

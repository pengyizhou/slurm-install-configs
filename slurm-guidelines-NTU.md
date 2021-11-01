# SLURM RUNNING GUIDE
*YIZHOU PENG* 

*MICL@NTU, 11-1-2021*

---

## SBATCH COMMAND
**We DO NOT use *#SBATCH* command at the top of the running scripts like below.**
```
#!/bin/bash
#SBATCH -o job.%j.out
#SBATCH -J myFirstJob
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
```
**sbatch** is used to submit a job script for later execution. The script will typically contain one or more srun commands to launch parallel tasks.

### <font color="#4b8bf4">***sbatch** scripts submit example.*</font><br/>
```
#!/bin/bash

steps=1-4
log=$something.log
cmd="slurm.pl --quiet"
nodelist=node01 # nodelist should always be one node only
# if you want to submit your jobs to one of multiple nodes
exclude=node0[2,4,6]

sbatch -o $log -w $nodelist $something.sh --steps $steps --cmd "$cmd"
# sbatch -o $log -x $exclude $something.sh --steps $steps --cmd "$cmd"
```
<font color="#dc4c3f">If you want to allocate GPUs, see the following usage guidelines for slurm.pl. DO NOT allocate GPU resources using **sbatch** command. </font><br/>

---

## SLURM.PL USAGE

+ Basic usage of slurm.pl
    ```
    nodelist=node01   # specify one node for running your jobs
    # DO NOT specify more than one node in nodelist

    num_threads_per_job=2   # specify number of CPUs for one srun job
    exclude_nodelist=node0[4-7]   # exclude nodes for submitting jobs
    memory_limit=20G   # srun jobs would be killed at the moment exceeded this memory limit
    ngpu=2   # number of GPUs you scripts needed
    log=$somelog.log
    
    slurm.pl --quiet --nodelist=$nodelist --gpu $ngpu\
         --num-threads $num_threads_per_job --mem $memory_limit \
         $log $scripts(shell, python, etc.) [ paramlist]

    # slurm.pl --quiet --exclude=$exclude_nodelist --gpu $ngpu\
         --num-threads $num_threads_per_job --mem $memory_limit \
         $log $scripts(shell, python, etc.) [ paramlist]
    ```
+ <font color="#4b8bf4">*Following is the slurm.pl example file used in our cluster.*</font><br/>

    ```
    #### slurm.pl used in our cluster
    #!/usr/bin/perl -w

    # In general, doing 
    # slurm.pl some.log a b c 
    # is like running the command a b c as an interactive SLURM job, and putting the 
    # standard error and output into some.log.
    # It is a de-facto-mimicry of run.pl, with the difference, that it allocates the
    # jobs on a slurm cluster, using SLURM's salloc.
    #
    # To run parallel jobs (backgrounded on the host machine), you can do (e.g.)
    # slurm.pl JOB=1:4 some.JOB.log a b c JOB is like running the command a b c JOB
    # and putting it in some.JOB.log, for each one. [Note: JOB can be any identifier].
    # If any of the jobs fails, this script will fail.

    # A typical example is:
    #  slurm.pl some.log my-prog "--opt=foo bar" foo \|  other-prog baz
    # and slurm.pl will run something like:
    # ( my-prog '--opt=foo bar' foo |  other-prog baz ) >& some.log
    # 
    # Basically it takes the command-line arguments, quotes them
    # as necessary to preserve spaces, and evaluates them with bash.
    # In addition it puts the command line at the top of the log, and
    # the start and end times of the command at the beginning and end.
    # The reason why this is useful is so that we can create a different
    # version of this program that uses a queueing system instead.
    #
    # You can also specify command-line options that are passed to SLURM's
    # salloc, e.g.:
    #
    # slurm.pl -p long -c 4 --mem=16g JOB=1:4 some.JOB.log a b c JOB
    #
    # The options "-p long -c 4 --mem=16g" are passed to salloc. In general, all
    # options of form "-foo something" and "--fii" (and also "-V") are recognized
    # as options that must be passed to salloc (yes, this is a bit hacky).
    #
    # In addition to that, the script also converts an option like '-pe smp N'
    # to a form '-c N'. This is because some Kaldi's scripts use this SGE syntax
    # to specify the number of CPU cores for a job.
    #

    @ARGV < 2 && die "usage: slurm.pl log-file command-line arguments..., hahahah";

    $jobstart=1;
    $jobend=1;
    $queue_opts = "";

    # First parse an option like JOB=1:4

    for ($x = 1; $x <= 3; $x++) { # This for-loop is to 
    # allow the JOB=1:n option to be interleaved with the
    # options to qsub.
    while (@ARGV >= 2 && $ARGV[0] =~ m:^-:) {
        $switch = shift @ARGV;
        if ($switch =~ m:--max-jobs-run:) {
            $queue_opts .=" --ntasks-per-node=7";
            print "queue_opts: $queue_opts\n";
        shift @ARGV;
        } elsif ($switch =~ m:--gpu:) {
        my $x = shift @ARGV;
        $queue_opts .= " --gres=gpu:$x";
        } elsif ($switch =~ m:--mem:) {
        my $mem_size = lc shift @ARGV;
        $mem_size =~ m:(\d+)(\S+):g or die;
        $mem_size = $1;  $mem_size="80" if ($mem_size < 22);
        $mem_size .= "g";
        $queue_opts .= " --mem=$mem_size";
        print "queue_opts: $queue_opts\n";
        } elsif ($switch =~ m:--num-threads:) {
        $nof_threads = shift @ARGV;
        $queue_opts .= " -c $nof_threads ";
        } elsif ($switch =~ m:--:) {  # e.g. --gres=gpu:1
        $queue_opts .= " $switch ";
        } elsif ($switch eq "-V") {
        $queue_opts .= " -V ";
        } else {
        $option = shift @ARGV;
        # Override '-pe smp 5' like option for SGE
        # for SLURM, the equivalent is '-c 5'.
        # This option is hard-coded in some training scripts
        if ($switch eq "-pe" && $option eq "smp") { # e.g. -pe smp 5
            $option2 = shift @ARGV;
            $nof_threads = $option2;
            $queue_opts .= "-c $nof_threads ";
            print STDERR "slurm.pl: converted SGE option '-pe smp $nof_threads' to SLURM option '-c $nof_threads'\n"
        } else {  
            $queue_opts .= "$switch $option ";
        }
        }
        }
    if ($ARGV[0] =~ m/^([\w_][\w\d_]*)+=(\d+):(\d+)$/) {
        $jobname = $1;
        $jobstart = $2;
        $jobend = $3;
        shift;
        if ($jobstart > $jobend) {
        die "queue.pl: invalid job range $ARGV[0]";
        }
        if ($jobstart <= 0) {
        die "run.pl: invalid job range $ARGV[0], start must be strictly positive (this is required for GridEngine compatibility).";                   
        }
    } elsif ($ARGV[0] =~ m/^([\w_][\w\d_]*)+=(\d+)$/) { # e.g. JOB=1.
        $jobname = $1;
        $jobstart = $2;
        $jobend = $2;
        shift;
    } elsif ($ARGV[0] =~ m/.+\=.*\:.*$/) {
        print STDERR "Warning: suspicious first argument to queue.pl: $ARGV[0]\n";                                                                      
    }
    }

    $logfile = shift @ARGV;

    if (defined $jobname && $logfile !~ m/$jobname/ &&
        $jobend > $jobstart) {
    print STDERR "slurm.pl: you are trying to run a parallel job but "
        . "you are putting the output into just one log file ($logfile)\n";
    exit(1);
    }

    #
    # Work out the command; quote escaping is done here.
    # Note: the rules for escaping stuff are worked out pretty
    # arbitrarily, based on what we want it to do.  Some things that
    # we pass as arguments to queue.pl, such as "|", we want to be
    # interpreted by bash, so we don't escape them.  Other things,
    # such as archive specifiers like 'ark:gunzip -c foo.gz|', we want
    # to be passed, in quotes, to the Kaldi program.  Our heuristic
    # is that stuff with spaces in should be quoted.  This doesn't
    # always work.
    #
    $cmd = "";
    foreach $x (@ARGV) {
    if ($x =~ m/^\S+$/) { $cmd .= $x . " "; } # If string contains no spaces, take                                                                    
                                                # as-is.
    elsif ($x =~ m:\":) { $cmd .= "'\''$x'\'' "; } # else if no dbl-quotes, use single                                                                
    else { $cmd .= "\"$x\" "; }  # else use double.
    }

    for ($jobid = $jobstart; $jobid <= $jobend; $jobid++) {
    $childpid = fork();
    if (!defined $childpid) { die "Error forking in slurm.pl (writing to $logfile)"; }                                                                
    if ($childpid == 0) { # We're in the child... this branch
        # executes the job and returns (possibly with an error status).
        if (defined $jobname) {
        $cmd =~ s/$jobname/$jobid/g;
        $logfile =~ s/$jobname/$jobid/g;
        }
        system("mkdir -p `dirname $logfile` 2>/dev/null");
        open(F, ">$logfile") || die "Error opening log file $logfile";
        print F "# " . $cmd . "\n";
        $startdate=`date`;
        chop $startdate;
        print F "# Invoked at " . $startdate . " from " . `hostname`;
        $starttime = `date +'%s'`;
        print F "#\n";
        close(F);

        # Pipe into bash.. make sure we're not using any other shell.
        # print STDERR "## LOG ($0): queue_opts= $queue_opts\n";
        open(B, "|-", "salloc $queue_opts srun $queue_opts bash") || die "Error opening shell command";                                                 
        print B "echo '#' Started at `date` on `hostname` 2>>$logfile >> $logfile;";                                                                    
        print B "( " . $cmd . ") 2>>$logfile >> $logfile";
        close(B);                   # If there was an error, exit status is in $?                                                                       
        $ret = $?;

        $endtime = `date +'%s'`;
        open(F, ">>$logfile") || die "Error opening log file $logfile (again)";                                                                         
        $enddate = `date`;
        chop $enddate;
        print F "# Ended (code $ret) at " . $enddate . ", elapsed time " . ($endtime-$starttime) . " seconds\n";                                        
        close(F);
        exit($ret == 0 ? 0 : 1);
    }
    }
    $ret = 0;
    $numfail = 0;
    for ($jobid = $jobstart; $jobid <= $jobend; $jobid++) {
    $r = wait();
    if ($r == -1) { die "Error waiting for child process"; } # should never happen.                                                                   
    if ($? != 0) { $numfail++; $ret = 1; } # The child process failed.
    }

    if ($ret != 0) {
    $njobs = $jobend - $jobstart + 1;
    if ($njobs == 1) {
        print STDERR "slurm.pl: job failed, log is in $logfile\n";
        if ($logfile =~ m/JOB/) {
        print STDERR "slurm.pl: probably you forgot to put JOB=1:\$nj in your script.\n";                                                             
        }
    }
    else {
        $logfile =~ s/$jobname/*/g;
        print STDERR "slurm.pl: $numfail / $njobs failed, log is in $logfile\n";                                                                        
    }
    }

    exit ($ret);
    ```

* <big>For **kaldi** scripts</big>



    Kaldi format scripts uses **cmd** to run distributed/parallel jobs. So it is easy to modify.

    Kaldi format scripts would automatically add GPU allocate parameters, so we don't need to add GPU related parameters in the **cmd**.
    ```
    #### cmd.sh at project folder
    export cmd="slurm.pl --quiet"
    export decode_cmd="slurm.pl --quiet --exclude=node0[4-7]"
    export train_cmd="slurm.pl --quiet --exclude=node0[1-2,4-9] --num-threads 2"
    ```

    And source the cmd.sh in your running scripts.

    ```
    #!/bin/bash
    nj=10
    steps=
    gmm_stage=
    train_stage=-10
    egs_stage=-100
    chain_tree_pdfs=10000
    epoch=
    ###------------------------------------------------
    # begin option
    frames_per_iter=10000000
    xent_regularize=0.1
    dropout_schedule='0,0@0.20,0.5@0.50,0'
    # training chunk-options
    chunk_width=140,100,160
    # training options
    srand=0
    remove_egs=false
    common_egs_dir=""
    # end option
    echo 
    echo "$0 $@"
    echo
    set -e

    . path.sh
    . cmd.sh
    . parse_options.sh || { echo "parse_options.sh expected ..." && exit 1; }
    ```
+ For python scripts

    For most of the python scripts, we should specify which GPUs the scripts would use. So we would mention an environment variable  **CUDA_VISIBLE_DEVICES** here.

    <font color="#dc4c3f">In the scripts, we should **NEVER** export the variable  **CUDA_VISIBLE_DEVICES** manually, like ``` export CUDA_VISIBLE_DEVICES="1,2,3" ```, and **NEVER** specify the GPU id for your scripts like  </font><br/>

    ``` 
    gpu_id="2"
    python $somewhere/train.py --gpu $gpu_id [--paramlist] 
    # here --gpu means the id of GPUs it will allocate, not the number of GPUs
    ```

    Following are some tips for python training scripts using slurm to allocate GPU resources.


    1. Move your python scripts needed GPU resources to another shell script from the one submitted by sbatch. Here I use the WeNet train script as an example. 
        ```
        #### train.sh
        #!/bin/bash
        ## some params

        . tools/parse_options.sh || exit 1;
        ## some params

        gpu_id=$CUDA_VISIBLE_DEVICES

        python wenet/bin/train.py --gpu $gpu_id \
            --config $train_config \
            --data_type raw \
            --symbol_table $dict \
            --train_data $train \
            --cv_data $dev \
            ${checkpoint:+--checkpoint $checkpoint} \
            --model_dir $dir \
            --ddp.init_method $init_method \
            --ddp.world_size $ngpu \
            --ddp.rank $[ part-1 ] \
            --ddp.dist_backend $dist_backend \
            --num_workers 1 \
            $cmvn_opts \
            --pin_memory || exit 1;
        ```

        ``` gpu_id=$CUDA_VISIBLE_DEVICES ``` where **CUDA_VISIBLE_DEVICES** is automatically set by **slurm** in the running nodes. So the script uses the GPUs slurm allocates for it.

       <font color="#dc4c3f">**WARNING**：If you set **CUDA_VISIBLE_DEVICES** yourself, you would be probably using the wrong GPUs and other users can still submit jobs to the GPUs you were using. This may crash both of the jobs or seriously affects the efficiency of the machine.  </font><br/>

        
    2. Edit the shell script where called the above script submitted by sbatch. 

        + The following example is for single thread training scripts like WeNet, where srun only allocate 1 GPU for each python training thread. 
        
            If multiple GPUs are needed because of large amount of training data or some other reasons, you can follow the Kaldi style distributed training scripts like below, which will start 2 srun jobs, each allocate 2 CPUs and 1 GPU.
            ```
            num_gpus=2
            # cmd="slurm.pl --quiet --nodelist=node01"
            $cmd --num-threads 2 --gpu 1 JOB=1:$num_gpus \
                $dir/log/train.JOB.log \
                train.sh \
                --train_config $train_config \
                --dict $dict \
                --train ./data/$train_set/data.list \
                --dev ./data/dev/data.list \
                --ngpu $num_gpus \
                --part JOB \
                --dist_backend $dist_backend \
                --checkpoint "$checkpoint" \
                --cmvn_dir ./data/$train_set/global_cmvn \
                $dir $init_method || exit 1;
            ```
        + If your python scripts are multi-threads style, you can allocate multiple GPUs with one srun job.
            ```
            num_gpus=2
            threads=$[ num_gpus*2 ]
            # cmd="slurm.pl --quiet --nodelist=node01"
            $cmd --num-threads $threads --gpu $num_gpus \
                $dir/log/train.log \
                train.sh \
                --train_config $train_config \
                --dict $dict \
                --train ./data/$train_set/data.list \
                --dev ./data/dev/data.list \
                --ngpu $num_gpus \
                --part $something \
                --dist_backend $dist_backend \
                --checkpoint "$checkpoint" \
                --cmvn_dir ./data/$train_set/global_cmvn \
                $dir $init_method || exit 1;
            ```


    <big>**NOTICE**</big>
    
    While submit GPU jobs, you would better set ```--num-threads 2*$num_gpus``` for both **Kaldi** or **Python** scripts which can help a lot for the efficiency use of GPUs.
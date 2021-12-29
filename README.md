# INSTALL
Use apt to install the necessary packages:

    sudo apt install -y slurm-wlm slurm-wlm-doc

Load file:///usr/share/doc/slurm-wlm/html/configurator.html in a browser (or file://wsl%24/Ubuntu/usr/share/doc/slurm-wlm/html/configurator.html on WSL2), and:

---
1. Set your machine's hostname in `SlurmctldHost` and `NodeName`.
2. Set `CPUs` as appropriate, and optionally `Sockets`, `CoresPerSocket`, and `ThreadsPerCore`. Use command `lscpu` to find what you have.
3. Set `RealMemory` to the number of megabytes you want to allocate to Slurm jobs,
4. Set `StateSaveLocation` to `/var/spool/slurm-llnl`.
5. Set `ProctrackType` to `linuxproc` because processes are less likely to escape Slurm control on a single machine config.
6. Make sure `SelectType` is set to `Cons_res`, and set `SelectTypeParameters` to `CR_Core_Memory`.
7. Set `JobAcctGatherType` to `Linux` to gather resource use per job, and set `AccountingStorageType` to `FileTxt`.

Hit `Submit`, and save the resulting text into `/etc/slurm-llnl/slurm.conf` i.e. the configuration file referred to in `/lib/systemd/system/slurmctld.service` and `/lib/systemd/system/slurmd.service`.

Load `/etc/slurm-llnl/slurm.conf` in a text editor, uncomment `DefMemPerCPU`, and set it to `8192` or whatever number of megabytes you want each job to request if not explicitly requested using `--mem` during job submission. [Read the docs](https://slurm.schedmd.com/slurm.conf.html) and edit other defaults as you see fit.

---

OR

---

Create two config files names `/etc/slurm-llnl/slurm.conf` and  `/etc/slurm-llnl/gres.conf`
  ```
  ## /etc/slurm-llnl/gres.conf
  ## config the gpus based on the server
  # This section of this file was automatically generated by cmd. Do not edit manually!
  # BEGIN AUTOGENERATED SECTION -- DO NOT REMOVE
  Name=gpu File=/dev/nvidia0
  Name=gpu File=/dev/nvidia1
  Name=gpu File=/dev/nvidia2
  Name=gpu File=/dev/nvidia3
  Name=gpu File=/dev/nvidia4
  Name=gpu File=/dev/nvidia5
  Name=gpu File=/dev/nvidia6
  Name=mic Count=0
  # END AUTOGENERATED SECTION   -- DO NOT REMOVE
  ```
  
  Only need to check the key: `SlurmctldHost` `NodeName` `PartitionName`, make them the correct config of the current server.
  
  ```
  ## /etc/slurm-llnl/slurm.conf
  # slurm.conf file generated by configurator.html.
  # Put this file on all nodes of your cluster.
  # See the slurm.conf man page for more information.
  #
  SlurmctldHost=xju-aslp4
  #SlurmctldHost=
  #
  #DisableRootJobs=NO
  #EnforcePartLimits=NO
  #Epilog=
  #EpilogSlurmctld=
  #FirstJobId=1
  #MaxJobId=999999
  GresTypes=gpu
  #GroupUpdateForce=0
  #GroupUpdateTime=600
  #JobFileAppend=0
  #JobRequeue=1
  #JobSubmitPlugins=1
  #KillOnBadExit=0
  #LaunchType=launch/slurm
  #Licenses=foo*4,bar
  #MailProg=/bin/mail
  #MaxJobCount=5000
  #MaxStepCount=40000
  #MaxTasksPerNode=128
  MpiDefault=none
  #MpiParams=ports=#-#
  #PluginDir=
  #PlugStackConfig=
  #PrivateData=jobs
  ProctrackType=proctrack/linuxproc
  #Prolog=
  #PrologFlags=
  #PrologSlurmctld=
  #PropagatePrioProcess=0
  #PropagateResourceLimits=
  #PropagateResourceLimitsExcept=
  #RebootProgram=
  ReturnToService=1
  #SallocDefaultCommand=
  SlurmctldPidFile=/var/run/slurmctld.pid
  SlurmctldPort=6817
  SlurmdPidFile=/var/run/slurmd.pid
  SlurmdPort=6818
  SlurmdSpoolDir=/var/spool/slurmd
  SlurmUser=slurm
  #SlurmdUser=root
  #SrunEpilog=
  #SrunProlog=
  StateSaveLocation=/var/spool/slurm-llnl
  SwitchType=switch/none
  #TaskEpilog=
  TaskPlugin=task/affinity
  TaskPluginParam=Sched
  #TaskProlog=
  #TopologyPlugin=topology/tree
  #TmpFS=/tmp
  #TrackWCKey=no
  #TreeWidth=
  #UnkillableStepProgram=
  #UsePAM=0
  #
  #
  # TIMERS
  #BatchStartTimeout=10
  #CompleteWait=0
  #EpilogMsgTime=2000
  #GetEnvTimeout=2
  #HealthCheckInterval=0
  #HealthCheckProgram=
  InactiveLimit=0
  KillWait=30
  #MessageTimeout=10
  #ResvOverRun=0
  MinJobAge=300
  #OverTimeLimit=0
  SlurmctldTimeout=120
  SlurmdTimeout=300
  #UnkillableStepTimeout=60
  #VSizeFactor=0
  Waittime=0
  #
  #
  # SCHEDULING
  #DefMemPerCPU=
  #MaxMemPerCPU=0
  #SchedulerTimeSlice=30
  SchedulerType=sched/backfill
  SelectType=select/cons_res
  SelectTypeParameters=CR_CPU
  #
  #
  # JOB PRIORITY
  #PriorityFlags=
  #PriorityType=priority/basic
  #PriorityDecayHalfLife=
  #PriorityCalcPeriod=
  #PriorityFavorSmall=
  #PriorityMaxAge=
  #PriorityUsageResetPeriod=
  #PriorityWeightAge=
  #PriorityWeightFairshare=
  #PriorityWeightJobSize=
  #PriorityWeightPartition=
  #PriorityWeightQOS=
  #
  #
  # LOGGING AND ACCOUNTING
  #AccountingStorageEnforce=0
  #AccountingStorageHost=
  #AccountingStorageLoc=
  #AccountingStoragePass=
  #AccountingStoragePort=
  AccountingStorageType=accounting_storage/filetxt
  #AccountingStorageUser=
  AccountingStoreJobComment=YES
  ClusterName=cluster
  #DebugFlags=
  #JobCompHost=
  #JobCompLoc=
  #JobCompPass=
  #JobCompPort=
  JobCompType=jobcomp/none
  #JobCompUser=
  #JobContainerType=job_container/none
  JobAcctGatherFrequency=30
  JobAcctGatherType=jobacct_gather/linux
  SlurmctldDebug=info
  #SlurmctldLogFile=
  SlurmdDebug=info
  #SlurmdLogFile=
  #SlurmSchedLogFile=
  #SlurmSchedLogLevel=
  #
  #
  # POWER SAVE SUPPORT FOR IDLE NODES (optional)
  #SuspendProgram=
  #ResumeProgram=
  #SuspendTimeout=
  #ResumeTimeout=
  #ResumeRate=
  #SuspendExcNodes=
  #SuspendExcParts=
  #SuspendRate=
  #SuspendTime=
  #
  #
  # COMPUTE NODES
  NodeName=xju-aslp4 CPUs=64 RealMemory=256000 Sockets=2 CoresPerSocket=32 ThreadsPerCore=1 Gres=gpu:7
  PartitionName=XJU Nodes=xju-aslp4 Default=YES MaxTime=INFINITE State=UP
  ```
  
  
  
  
---


Create `/var/spool/slurm-llnl` and `/var/log/slurm_jobacct.log`, then set ownership appropriately:

    sudo mkdir -p /var/spool/slurm-llnl
    sudo touch /var/log/slurm_jobacct.log
    sudo chown slurm:slurm /var/spool/slurm-llnl /var/log/slurm_jobacct.log

Install `mailutils` so that Slurm won't complain about `/bin/mail` missing:

    sudo apt install -y mailutils

Make sure munge is installed and running, and a `munge.key` was created with user-only read-only permissions, owned by `munge:munge`:

    sudo service munge start
    sudo ls -l /etc/munge/munge.key

Start services `slurmctld` and `slurmd`:

    sudo service slurmd start
    sudo service slurmctld start
    sudo systemctl enable slurmctld

Slurm management
  
  ```
  # FOR RESTART
  sudo systemctl restart slurmd 
  sudo systemctl restart slurmctld
  # FOR UNDRAIN
  sudo scontrol update NodeName=xju-aslp4 State=Down Reason="undrain"
  sudo scontrol update NodeName=xju-aslp4 State=RESUME
  # check node status
  scontrol show node $nodename
  # check job status
  squeue
  scontrol show job $jobid
  # show all node stauts
  sinfo
  ```
  
  ## REFERENCE

    https://slurm.schedmd.com/slurm.conf.html#OPT_SelectType

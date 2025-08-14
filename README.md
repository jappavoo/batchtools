#  GPU batch job utilities

## brun (br) create and run a command line as a GPU batch job

``` bash
   gun [-h] <command line>
     gun creates and submits a batch job to a GPU batch queue.  The arguments are treated as a bash command line
     that will execute as the batch job.  The behaviour of the job submission can be controlled via several
     environment variables.  Please, feel free to improve this documentation and fix 
     bugs ;-). By default the files and subdirectories of your current working directory form a context for 
     the job.  The context is copied to the container in which the batch job will execute.  Thus the commandline
     can reference files in the directory.  Additinoally, files created in the working directory of commandline
     in the container will be copied back so that you can inspect output of your job (eg. profiles, logs and outputs).
     These files and directories are placed in a directory make in job specific subdirectory of a directory called jobs.
     Eg.

1. The simple usage run a binary that exsits in the current directory on the default GPU type
$ br ./hello
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
RUNDIR: jobs/job-v100-9215

Note by default gun waits for the job to complete and displays after the standard output and error of the command line.
After that it display the directory where the outputs for the job where copied.

$ ls jobs/job-v100-9215/
job-v100-9215.log

2. Any additional files created in the remote copy of the context directory are copied to the job output directory
$ br "./hello | tee hello.log"
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
RUNDIR: jobs/job-v100-9477

$ ls jobs/job-v100-9215
job-v100-9215.log
csw-dev-0.test> ls jobs/job-v100-9477/
hello.log  job-v100-9477.log


3. You can set the NAME varible to provide a unique name for the job to help organized your runs.  Eg.

$ NAME=hello-run0 br "./hello | tee hello.log"
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
RUNDIR: jobs/hello-run0-v100-9753

$ ls jobs/hello-run0-v100-9753/
hello.log  hello-run-v100-9753.log


4. You can set VERBOSE=1 to see details of what gun is doing (for even more details set DEBUG=1)

$ VERBOSE=1 br "./hello | tee hello.log"
GPU:v100 JOB:job-v100-10006 CONTAINER:job-v100-10006-container QUEUE:v100-localqueue MAX_SEC:900 WAIT:1 JOB_WAIT: JOB_DELETE:1 CONTEXT_DIR:/opt/app-root/src/cudasudawuda/content/src OUTPUT_DIR=/opt/app-root/src/cudasudawuda/content/src/jobs/job-v100-10006
CMD: ./hello | tee hello.log
apiVersion: batch/v1
kind: Job
metadata:
  name: job-v100-10006
  labels:
    kueue.x-k8s.io/queue-name: v100-localqueue
    test_name: kueue_test
spec:
  parallelism: 1
  completions: 1
  backoffLimit: 0
  template:
    spec:
      restartPolicy: Never
      maximumExecutionTimeSeconds: 900
      containers:
        - name: job-v100-10006-container
          image: image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/csw-run-f25:latest
          command: ["/bin/sh", 
                    "-c", 
                    "export RSYNC_RSH='oc rsh -c csw-dev';
                     mkdir job-v100-10006 &&
                     rsync  --archive --no-owner --no-group --omit-dir-times --numeric-ids csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/jobs/job-v100-10006/getlist job-v100-10006/getlist >/dev/null 2>&1 &&
                     rsync   -r --archive --no-owner --no-group --omit-dir-times --numeric-ids --files-from=job-v100-10006/getlist csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/ job-v100-10006/ &&
                     find job-v100-10006 -mindepth 1 -maxdepth 1 > job-v100-10006/gotlist &&
                     cd job-v100-10006 && ./hello | tee hello.log |& tee job-v100-10006.log; cd ..;
                     rsync  --archive --no-owner --no-group --omit-dir-times --no-relative --numeric-ids --exclude-from=job-v100-10006/gotlist job-v100-10006 csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/jobs"]
          resources:
            requests:
              nvidia.com/gpu: 1
            limits:
              nvidia.com/gpu: 1
job.batch/job-v100-10006 created
Job Created: job-v100-10006
Name:             job-v100-10006
Namespace:        ja-ope-test
Selector:         batch.kubernetes.io/controller-uid=0894f77d-05a2-4cc0-bc9d-f875514fe2df
Labels:           kueue.x-k8s.io/queue-name=v100-localqueue
                  test_name=kueue_test
Annotations:      <none>
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Suspend:          false
Backoff Limit:    0
Start Time:       Thu, 14 Aug 2025 04:48:39 +0000
Pods Statuses:    1 Active (0 Ready) / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  batch.kubernetes.io/controller-uid=0894f77d-05a2-4cc0-bc9d-f875514fe2df
           batch.kubernetes.io/job-name=job-v100-10006
           controller-uid=0894f77d-05a2-4cc0-bc9d-f875514fe2df
           job-name=job-v100-10006
           kueue.x-k8s.io/podset=main
  Containers:
   job-v100-10006-container:
    Image:      image-registry.openshift-image-registry.svc:5000/redhat-ods-applications/csw-run-f25:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
      export RSYNC_RSH='oc rsh -c csw-dev'; mkdir job-v100-10006 && rsync  --archive --no-owner --no-group --omit-dir-times --numeric-ids csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/jobs/job-v100-10006/getlist job-v100-10006/getlist >/dev/null 2>&1 && rsync   -r --archive --no-owner --no-group --omit-dir-times --numeric-ids --files-from=job-v100-10006/getlist csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/ job-v100-10006/ && find job-v100-10006 -mindepth 1 -maxdepth 1 > job-v100-10006/gotlist && cd job-v100-10006 && ./hello | tee hello.log |& tee job-v100-10006.log; cd ..; rsync  --archive --no-owner --no-group --omit-dir-times --no-relative --numeric-ids --exclude-from=job-v100-10006/gotlist job-v100-10006 csw-dev-0:/opt/app-root/src/cudasudawuda/content/src/jobs
    Limits:
      nvidia.com/gpu:  1
    Requests:
      nvidia.com/gpu:  1
    Environment:       <none>
    Mounts:            <none>
  Volumes:             <none>
  Node-Selectors:      <none>
  Tolerations:         nvidia.com/gpu.product=Tesla-V100-PCIE-32GB:NoSchedule
Events:
  Type    Reason            Age   From                        Message
  ----    ------            ----  ----                        -------
  Normal  Suspended         0s    job-controller              Job suspended
  Normal  CreatedWorkload   0s    batch/job-kueue-controller  Created Workload: ja-ope-test/job-job-v100-10006-285b0
  Normal  Started           0s    batch/job-kueue-controller  Admitted by clusterQueue v100-clusterqueue
  Normal  SuccessfulCreate  0s    job-controller              Created pod: job-v100-10006-88lpd
  Normal  Resumed           0s    job-controller              Job resumed
Waiting job-v100-10006 for condition: complete or failed
job.batch/job-v100-10006 condition met
job-v100-10006: Completed
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
job.batch "job-v100-10006" deleted
RUNDIR: jobs/job-v100-10006


5. If you don't want to wait for the job to complete set WAIT=0.  Note when WAIT=1 the job will be deleted
   automatically for you by brun.  If you set WAIT=0 you will need manage the job.  Eg monitor it and once
   you know it has finished delete.  See 'bjobs' (list the status of your current gpu jobs), 
   'gqstat' (inspect cluster wide gpu queues -- show their status),  'bps' (list all running gpu  pods on cluster nodes), 
   and 'bdel' (delete a gpu job)

$ WAIT=0 NAME=myasync-run0 gun "{ nvidia-smi; ./hello; sleep 15; }" 
WARNING: Not waiting for myasync-run0-v100-14109 to finish you must manually delete with 'gel myasync-run0-v100-14109'
myasync-run0-v100-14109 jobs/myasync-run0-v100-14109

$ bj
NAME                      STATUS    COMPLETIONS   DURATION   AGE
myasync-run0-v100-14109   Running   0/1           4s         4s

$ bq
a100-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
dummy-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
h100-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
v100-clusterqueue	 admitted: 1 pending: 0 reserved: 1 resources:3 BestEffortFIFO

$ bs
wrk-3: BUSY 1 ja-ope-test/myasync-run0-v100-14109-67k8c

$ gb
NAME                      STATUS    COMPLETIONS   DURATION   AGE
myasync-run0-v100-14109   Running   0/1           23s        23s

$ bq
a100-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
dummy-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
h100-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:0 BestEffortFIFO
v100-clusterqueue	 admitted: 0 pending: 0 reserved: 0 resources:3 BestEffortFIFO

$ bj
NAME                      STATUS     COMPLETIONS   DURATION   AGE
myasync-run0-v100-14109   Complete   1/1           26s        32s

$ bs

$ cat jobs/myasync-run0-v100-14109/myasync-run0-v100-14109.log 
Thu Aug 14 05:36:25 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.148.08             Driver Version: 570.148.08     CUDA Version: 12.8     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla V100-PCIE-32GB           On  |   00000000:3B:00.0 Off |                    0 |
| N/A   35C    P0             25W /  250W |       0MiB /  32768MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU

$ bd myasync-run0-v100-14109
job.batch "myasync-run0-v100-14109" deleted


6. Note the tool chain is available in the batch pod

$ br "make hello && ./hello | tee hello.log"
nvcc  hello.cu -o hello
Hello from CPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
Hello from GPU
RUNDIR: jobs/job-v100-10422
$
 ```


## bjobs (bj) display the status of your current GPU batch jobs

## bqstat (bq)  display the status of the GPU batch queues

## bps (bs) list running GPU job pods cluster wide (all users) based on the node they are on

## bdel (bd) delete a GPU job

## bpods (bp) list pods of a GPU job

## bwait (bw)  wait for a GPU job to finish

Not tested 

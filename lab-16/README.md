# Lab 16 - Jobs / Cronjobs

## Jobs

A Job creates one or more Pods and ensures that a specified number of them 
successfully terminate. As pods successfully complete, the Job tracks the 
successful completions. When a specified number of successful completions is 
reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods 
it created.

A simple case is to create one Job object in order to reliably run one Pod to 
completion. The Job object will start a new Pod if the first Pod fails or is 
deleted (for example due to a node hardware failure or a node reboot).

You can also use a Job to run multiple Pods in parallel.

## Task 0: Creating a namespace

Create a namespace for this lab:

```
kubectl create ns lab-16

---

namespace "lab-16" created
```

## Task 1: Running an example Job

Here is an example Job config. It computes π to 2000 places and prints it out. 
It takes around 10s to complete.

lab-16-job.yml:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

Now let’s run `kubectl create -f lab-16-job.yml -n lab-16` to deploy the example.

## Task 2: Check the status of the job

```
kubectl describe jobs/pi -n lab-16

---

Name:           pi
Namespace:      default
Selector:       controller-uid=f08d7c7d-c8c4-11e9-b7ce-42010a84026a
Labels:         controller-uid=f08d7c7d-c8c4-11e9-b7ce-42010a84026a
                job-name=pi
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"pi","namespace":"default"},"spec":{"backoffLimit":4,"template":...
Parallelism:    1
Completions:    1
Start Time:     Tue, 27 Aug 2019 14:19:38 +0200
Completed At:   Tue, 27 Aug 2019 14:20:17 +0200
Duration:       39s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=f08d7c7d-c8c4-11e9-b7ce-42010a84026a
           job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  51s   job-controller  Created pod: pi-vxccp
```

To view completed Pods of a Job, use `kubectl get pods -n lab-16`.

To list all the Pods that belong to a Job in a machine readable form, you can use a command like this:

```
pods=$(kubectl get pods -n lab-16 --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
echo $pods

kubectl logs $pods -n lab-16

3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

### Job Termination and Cleanup

When a Job completes, no more Pods are created, but the Pods are not deleted 
either. Keeping them around allows you to still view the logs of completed pods 
to check for errors, warnings, or other diagnostic output. The job object also 
remains after it is completed so that you can view its status. It is up to the 
user to delete old jobs after noting their status. Delete the job with kubectl 
(e.g. kubectl delete jobs/pi or kubectl delete -f ./lab-16-job.yml). When you delete 
the job using kubectl, all the pods it created are deleted too.

## Cronjob

A Cron Job creates Jobs on a time-based schedule.

One CronJob object is like one line of a crontab (cron table) file. It runs a 
job periodically on a given schedule, written in Cron format.

## Task 3: Creating a Cron Job

Cron jobs require a config file. This example cron job config .spec file prints 
the current time and a hello message every minute:

Check out the following CronJob yaml:

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

Run the example CronJob by using this command:

```
kubectl create -f https://k8s.io/examples/application/job/cronjob.yaml -n lab-16
```

After creating the cron job, get its status using this command:

```
kubectl get cronjob hello -n lab-16
```

As you can see from the results of the command, the cron job has not scheduled 
or run any jobs yet. Watch for the job to be created in around one minute:

```
kubectl get jobs --watch -n lab-16
```

The output is similar to this:

```
NAME               COMPLETIONS   DURATION   AGE
hello-4111706356   0/1                      0s
hello-4111706356   0/1   0s    0s
hello-4111706356   1/1   5s    5s
```

Now, find the pods that the last scheduled job created and view the standard output of one of the pods.

```
# Replace "hello-4111706356" with the job name in your system
pods=$(kubectl get pods -n lab-16 --selector=job-name=hello-4111706356 --output=jsonpath={.items[].metadata.name})
```

Show pod log:

```
kubectl logs $pods -n lab-16

Tue Aug 27 12:58:07 UTC 2019
Hello from the Kubernetes cluster
```

Delete the Cron Job

```
kubectl delete cronjob hello -n lab-16
```

## Task 4: Cleaning up

In order to delete our daemonset use the following command:

```
kubectl delete ns lab-16

---

namespace "lab-16" deleted
```

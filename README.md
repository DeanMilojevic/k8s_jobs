<p align=center>
  <img alt="k8s" src="./resources/k8s.svg" />
</p>

#

K8s job creates one or more pods in the cluster that run till completion. That is, the k8s will ensure that they run and successfully finish the indented action. Tho, with CronJobs this can become a reoccurring thing based on some Cron schedule. Compared to the pods job just once and they can be configured to run in parallel. This is useful thing when doing some batch processing. When you delete a job, pod will be deleted as well. The successfully completed job are kept in the history.

## Declarative way of defining Jobs

```yml
apiVersion: batch/v1
kind: Job
spec:
  parallelism: 2          # how many parallel jobs we can run
  completions: 4          # in case we run job in parallel manner how many times we need to run the job to mark the job as completed
  template:
    metadata:
      name: time-teller
    spec:
      restartPolicy: Never
      containers:
      - image: alpine
        name: alpine
        command:
        - "sh"
        - "-c"
        - "echo job"
        resources:
          limits:
            memory: "64Mi"
            cpu: "100m"
```

The job defined above will just echo out `hello from a job`. Nothing special, but the point is not to demonstrate what you want to run within your job. Now let us continue and apply the job and see what is happening:

```bash
kubectl apply -f src/job.yaml
job.batch/time-teller created

kubectl get jobs time-teller -o yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"time-teller","namespace":"default"},"spec":{"completions":4,"parallelism":2,"template":{"metadata":{"name":"time-teller"},"spec":{"containers":[{"command":["sh","-c","echo job"],"image":"alpine","name":"alpine","resources":{"limits":{"cpu":"100m","memory":"64Mi"}}}],"restartPolicy":"Never"}}}}
  creationTimestamp: "2020-05-31T16:16:00Z"
  labels:
    controller-uid: b66c2abc-4774-4086-b3e7-b6bf0f14e4ca
    job-name: time-teller
  name: time-teller
  namespace: default
  resourceVersion: "344226"
  selfLink: /apis/batch/v1/namespaces/default/jobs/time-teller
  uid: b66c2abc-4774-4086-b3e7-b6bf0f14e4ca
spec:
  backoffLimit: 6
  completions: 4
  parallelism: 2
  selector:
    matchLabels:
      controller-uid: b66c2abc-4774-4086-b3e7-b6bf0f14e4ca
  template:
    metadata:
      creationTimestamp: null
      labels:
        controller-uid: b66c2abc-4774-4086-b3e7-b6bf0f14e4ca
        job-name: time-teller
      name: time-teller
    spec:
      containers:
      - command:
        - sh
        - -c
        - echo job
        image: alpine
        imagePullPolicy: Always
        name: alpine
        resources:
          limits:
            cpu: 100m
            memory: 64Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Never
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  completionTime: "2020-05-31T16:16:10Z"
  conditions:
  - lastProbeTime: "2020-05-31T16:16:10Z"
    lastTransitionTime: "2020-05-31T16:16:10Z"
    status: "True"
    type: Complete
  startTime: "2020-05-31T16:16:00Z"
  succeeded: 4
```

## Declarative way of defining CronJobs

Now let us create a CronJob.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: time-teller
spec:
  schedule: 1 * * * *
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - image: alpine
            name: alpine
            command:
            - "sh"
            - "-c"
            - "echo cronjob"
            resources:
              limits:
                memory: "64Mi"
                cpu: "100m"
```

```bash
kubectl apply -f src/cronjob.yaml
cronjob.batch/time-teller created

kubectl  get cronjobs.batch time-teller
NAME          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
time-teller   */1 * * * *   False     0        <none>          13s

k get cronjobs.batch time-teller -o yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"batch/v1beta1","kind":"CronJob","metadata":{"annotations":{},"name":"time-teller","namespace":"default"},"spec":{"concurrencyPolicy":"Forbid","jobTemplate":{"spec":{"template":{"spec":{"containers":[{"command":["sh","-c","echo `hello from a cronjob`"],"image":"alpine","name":"alpine","resources":{"limits":{"cpu":"100m","memory":"64Mi"}}}],"restartPolicy":"OnFailure"}}}},"schedule":"*/1 * * * *"}}
  creationTimestamp: "2020-05-31T16:32:55Z"
  name: time-teller
  namespace: default
  resourceVersion: "346259"
  selfLink: /apis/batch/v1beta1/namespaces/default/cronjobs/time-teller
  uid: 1994be8d-af95-4f1d-b257-ba87bc3fedf5
spec:
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  jobTemplate:
    metadata:
      creationTimestamp: null
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - sh
            - -c
            - echo `hello from a cronjob`
            image: alpine
            imagePullPolicy: Always
            name: alpine
            resources:
              limits:
                cpu: 100m
                memory: 64Mi
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: OnFailure
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
  schedule: '*/1 * * * *'
  successfulJobsHistoryLimit: 3
  suspend: false
status:
  lastScheduleTime: "2020-05-31T16:35:00Z"  
```

## Summary

And that is pretty much it about jobs/cronjobs. Nothing complicated yet a really powerful out of the box feature.

## Tools

[Cron tool](https://crontab.guru/)

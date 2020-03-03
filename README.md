# tekton-cat-misbehave

Towards a conference talk. Work very much in progress. 

## Setup

Install Tekton 0.10.1

## Summary ##

Lots of workings-out and logs in this doc, so it's a stack with latest info at the top. What we have: 

- March 2nd: an 'output' resource is enough to create an auto-pvc. 
- March 3rd: a 'from' clause, and file copying, are required when creating files in a workspace under a Git pipeline resource.

## March 3rd part 2. 
I have a multi-step pipeline that can git clone, and modify files. Let's assume I want to build with kaniko or buildah - next failure should be docker push. 



-------------------------


```
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.10.1/release.yaml
```

## March 3rd: 
Dynamic provisioning over. We know what causes it. nfs.sh https://raw.github.ibm.com/AROBERTS/ocp-on-fyre/master/nfs-storage-provisioner.sh?token=AAA23VmrJBoxJWVmcvmVhQ4cNQCkivXZks5eZ5OYwA%3D%3D fails with 

+ oc -n openshift-image-registry scale deploy image-registry --replicas=0
Error from server (NotFound): deployments.extensions "image-registry" not found

Fix up script by commenting out lines with image-registry in (near start and end) until it passes. 

NAME                           STATUS        VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
basic-pipeline-run-p6z5m-pvc   Terminating   pvc-82c44b0e-8067-4876-bb1d-ad697bf5a467   5Gi        RWO            managed-nfs-storage   54s

PipelineRun now executes as expected. Files in the git repo are correctly found by the second task. What about new files we create, or files we change?

1. Create a new file. 

Copy march-2 to march-3

Required:

In the Pipeline: from

    - name: second-task
      runAfter: 
        - git-clone
      taskRef: 
        name: task-2
      resources: 
        inputs: 
          - name: gitrepo
            resource: repo-in
            from: [git-clone]

Without the from, the files are checked out again clean in task-2

In task-1:

cp -r $(inputs.resources.gitrepo.path)/* $(outputs.resources.gitrepo.path)

Without the cp. no files appear in /workspace/gitrepo in task-2

So 'from' and 'cp' are both required to link files from two tasks. 






## March 2nd: redo dynamic provisioning investigations

-- Get a 2-stage pipeline using Git PipelineResource. git clone / cat file
Fails, on Tekton 0.10.1, for lack of dynamic provisioning once you have:
- Two Tasks - one works fine
- A Git Pipeline Resource. Adding the `outputs` section generates the pvc
- Fails with one task of the form, 

apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: task-1
spec:
  inputs:
    resources:
      - name: gitrepo
        type: git
  outputs: 
    resources: 
      - name: gitrepo
        type: git
  steps:
    - name: hello
      image: ubuntu
      command: ["bash", "-ce"]
      args:
      - |
        echo "Hello from task-1"
        cat /workspace/gitrepo/readme.md

and a Pipeline, 

apiVersion: tekton.dev/v1alpha1
kind: Pipeline
metadata:
  name: basic-pipeline
spec:
  resources: 
    - name: repo-in
      type: git
  tasks:
    - name: git-clone
      taskRef: 
        name: task-1
      resources: 
        inputs:
          - name: gitrepo
            resource: repo-in
        outputs: 
          - name: gitrepo
            resource: repo-in

k get pods
NAME                                                   READY   STATUS      RESTARTS   AGE
basic-pipeline-run-6f92p-git-clone-w9l9t-pod-sd79n     0/5     Pending     0          113s

k get pod basic-pipeline-run-6f92p-git-clone-w9l9t-pod-sd79n -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"tekton.dev/v1alpha1","kind":"Task","metadata":{"annotations":{},"name":"task-1","namespace":"default"},"spec":{"inputs":{"resources":[{"name":"gitrepo","type":"git"}]},"outputs":{"resources":[{"name":"gitrepo","type":"git"}]},"steps":[{"args":["echo \"Hello from task-1\"\ncat /workspace/gitrepo/readme.md\n"],"command":["bash","-ce"],"image":"ubuntu","name":"hello"}]}}
    tekton.dev/release: devel
  creationTimestamp: "2020-03-03T11:16:52Z"
  labels:
    app.kubernetes.io/managed-by: tekton-pipelines
    tekton.dev/pipeline: basic-pipeline
    tekton.dev/pipelineRun: basic-pipeline-run-6f92p
    tekton.dev/pipelineTask: git-clone
    tekton.dev/task: task-1
    tekton.dev/taskRun: basic-pipeline-run-6f92p-git-clone-w9l9t
  name: basic-pipeline-run-6f92p-git-clone-w9l9t-pod-sd79n
  namespace: default
  ownerReferences:
  - apiVersion: tekton.dev/v1alpha1
    blockOwnerDeletion: true
    controller: true
    kind: TaskRun
    name: basic-pipeline-run-6f92p-git-clone-w9l9t
    uid: d45dd879-839c-4ea0-8eba-f3af57d65bf6
  resourceVersion: "2448868"
  selfLink: /api/v1/namespaces/default/pods/basic-pipeline-run-6f92p-git-clone-w9l9t-pod-sd79n
  uid: efbdd0c8-26c5-4293-94f4-ea06edc32024
spec:
  containers:
  - args:
    - -wait_file
    - /tekton/downward/ready
    - -wait_file_content
    - -post_file
    - /tekton/tools/0
    - -termination_path
    - /tekton/termination
    - -entrypoint
    - mkdir
    - --
    - -p
    - /workspace/output/gitrepo
    command:
    - /tekton/tools/entrypoint
    env:
    - name: HOME
      value: /tekton/home
    image: busybox
    imagePullPolicy: Always
    name: step-create-dir-gitrepo-mcmzt
    resources:
      requests:
        cpu: "0"
        ephemeral-storage: "0"
        memory: "0"
    terminationMessagePath: /tekton/termination
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /tekton/downward
      name: tekton-internal-downward
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
    workingDir: /workspace
  - args:
    - -wait_file
    - /tekton/tools/0
    - -post_file
    - /tekton/tools/1
    - -termination_path
    - /tekton/termination
    - -entrypoint
    - /ko-app/git-init
    - --
    - -url
    - https://github.com/mnuttall/simple.git
    - -revision
    - master
    - -path
    - /workspace/gitrepo
    command:
    - /tekton/tools/entrypoint
    env:
    - name: HOME
      value: /tekton/home
    - name: TEKTON_RESOURCE_NAME
      value: git-repo
    image: gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init@sha256:18ffa2bfc14b1fa6d39f62271beacf6bbc38e7cd2e255184dec477c2936270bc
    imagePullPolicy: IfNotPresent
    name: step-git-source-git-repo-x4hqk
    resources:
      requests:
        cpu: "0"
        ephemeral-storage: "0"
        memory: "0"
    terminationMessagePath: /tekton/termination
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
    workingDir: /workspace
  - args:
    - -wait_file
    - /tekton/tools/1
    - -post_file
    - /tekton/tools/2
    - -termination_path
    - /tekton/termination
    - -entrypoint
    - bash
    - --
    - -ce
    - |
      echo "Hello from task-1"
      cat /workspace/gitrepo/readme.md
    command:
    - /tekton/tools/entrypoint
    env:
    - name: HOME
      value: /tekton/home
    image: ubuntu
    imagePullPolicy: Always
    name: step-hello
    resources:
      requests:
        cpu: "0"
        ephemeral-storage: "0"
        memory: "0"
    terminationMessagePath: /tekton/termination
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
    workingDir: /workspace
  - args:
    - -wait_file
    - /tekton/tools/2
    - -post_file
    - /tekton/tools/3
    - -termination_path
    - /tekton/termination
    - -entrypoint
    - mkdir
    - --
    - -p
    - /pvc/git-clone/gitrepo
    command:
    - /tekton/tools/entrypoint
    env:
    - name: HOME
      value: /tekton/home
    image: busybox
    imagePullPolicy: Always
    name: step-source-mkdir-git-repo-pm2qs
    resources:
      requests:
        cpu: "0"
        ephemeral-storage: "0"
        memory: "0"
    terminationMessagePath: /tekton/termination
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /pvc
      name: basic-pipeline-run-6f92p-pvc
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
    workingDir: /workspace
  - args:
    - -wait_file
    - /tekton/tools/3
    - -post_file
    - /tekton/tools/4
    - -termination_path
    - /tekton/termination
    - -entrypoint
    - cp
    - --
    - -r
    - /workspace/output/gitrepo/.
    - /pvc/git-clone/gitrepo
    command:
    - /tekton/tools/entrypoint
    env:
    - name: HOME
      value: /tekton/home
    image: busybox
    imagePullPolicy: Always
    name: step-source-copy-git-repo-j2rtw
    resources:
      requests:
        cpu: "0"
        ephemeral-storage: "0"
        memory: "0"
    terminationMessagePath: /tekton/termination
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /pvc
      name: basic-pipeline-run-6f92p-pvc
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
    workingDir: /workspace
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  imagePullSecrets:
  - name: default-dockercfg-c5xnn
  initContainers:
  - args:
    - -docker-cfg=default-dockercfg-c5xnn
    command:
    - /ko-app/creds-init
    env:
    - name: HOME
      value: /tekton/home
    image: gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/creds-init@sha256:959b0d9a2d43d35e15a85460cc86567d308f467ee8ec16dbd9b32f51ce75d582
    imagePullPolicy: IfNotPresent
    name: credential-initializer
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /workspace
      name: tekton-internal-workspace
    - mountPath: /tekton/home
      name: tekton-internal-home
    - mountPath: /tekton/creds-secrets/default-dockercfg-c5xnn
      name: tekton-internal-secret-volume-default-dockercfg-c5xnn
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
  - command:
    - cp
    - /ko-app/entrypoint
    - /tekton/tools/entrypoint
    image: gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/entrypoint@sha256:cf0e81477c45dca0df6253e3239f6f0603700641292bf207503a7b267dc4c916
    imagePullPolicy: IfNotPresent
    name: place-tools
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /tekton/tools
      name: tekton-internal-tools
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-8nhf6
      readOnly: true
  priority: 0
  restartPolicy: Never
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: tekton-internal-workspace
  - emptyDir: {}
    name: tekton-internal-home
  - name: tekton-internal-secret-volume-default-dockercfg-c5xnn
    secret:
      defaultMode: 420
      secretName: default-dockercfg-c5xnn
  - emptyDir: {}
    name: tekton-internal-tools
  - downwardAPI:
      defaultMode: 420
      items:
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.annotations['tekton.dev/ready']
        path: ready
    name: tekton-internal-downward
  - name: basic-pipeline-run-6f92p-pvc
    persistentVolumeClaim:
      claimName: basic-pipeline-run-6f92p-pvc
  - name: default-token-8nhf6
    secret:
      defaultMode: 420
      secretName: default-token-8nhf6
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-03-03T11:16:52Z"
    message: pod has unbound immediate PersistentVolumeClaims (repeated 3 times)
    reason: Unschedulable
    status: "False"
    type: PodScheduled
  phase: Pending
  qosClass: BestEffort

 ~/dev/myghe/testBuild  k get pvc
NAME                           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
basic-pipeline-run-6f92p-pvc   Pending                                                     4m9s

/pvc is not created if the 'outputs' section is commented out. 

=> (Git) PipelineResource needs dynamic provisioning (or a pvc) if passing a workspace between more than one Task. 

From Tekton Slack:
https://github.com/tektoncd/pipeline/issues/1986
vdm confirmed that 1986 + https://github.com/tektoncd/catalog/blob/master/git/git-clone.yaml is the strategy, but 'we won't delete GitResource any time soon'. 


## Investigations - Feb, /feb-test, abandoned

1. Run without dynamic provisioning in place

-> 'manger' cluster
No NFS setup. 
https://github.com/tektoncd/pipeline/releases/download/v0.10.1/release.notags.yaml


```
kubectl apply -f config
tkn pipeline start buildah-build-pipeline -r git-source=simple-ghe -r docker-image=mark-simple-docker-image
```

ran up to 'no git credentials' - surprisingly! 
However, the pipeline contains only a single task. What about the 2-task one?
2-task fails at git checkout. change git url to github.com/mnuttall/simple
next: fails at docker push. comment that out for now. 

oc adm policy add-scc-to-user privileged -z [service_account_name] -n [namespace]

-- gets into task two, fails at lack of rbac to deploy. 

See 'file-copying' for some experiments with workspaces. emptyDir{} is not shared, and can't be hacked or mounted into two Tasks, so it's not a great 'first thing to go wrong.'

Suggest using https://github.com/tektoncd/pipeline/blob/master/examples/pipelineruns/output-pipelinerun.yaml to use a Git resource (e.g. for this repo in github.com) and then omit to write to /workspace/output. When trying to do this with workspaces we get, 

[write-something : write-new-stuff] cp: cannot create regular file '/workspace/output/scratch': No such file or directory

but there's nothing in 'workspaces' doc about 'output'. 'workspaces' aren't yet the obvious sucessor to Git resources 


2. Various flavours of credential and secret failures. 

2.1 No git credentials on the service account (no creds of any kind)

```
kubectl apply -f config
tkn pipeline start buildah-build-pipeline -r git-source=simple-ghe -r docker-image=mark-simple-docker-image
```

tkn pipeline start buildah-build-pipeline -r git-source=simple-ghe -r docker-image=mark-simple-docker-image
Pipelinerun started: buildah-build-pipeline-run-bt2q9
Showing logs...
[build-simple : create-dir-image-pnq6q] {"level":"info","ts":1582891063.25439,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"KO_DATA_PATH\" does not exist or is empty"}

[build-simple : git-source-simple-ghe-d55p5] {"level":"info","ts":1582891070.9629781,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: open /var/run/ko/refs/heads/master: no such file or directory"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"info","ts":1582891099.9626353,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: open /var/run/ko/refs/heads/master: no such file or directory"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"error","ts":1582891102.4176126,"logger":"fallback-logger","caller":"git/git.go:41","msg":"Error running git [fetch --recurse-submodules=yes --depth=1 origin master]: exit status 128\nfatal: could not read Username for 'https://github.ibm.com': No such device or address\n","stacktrace":"github.com/tektoncd/pipeline/pkg/git.run\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:41\ngithub.com/tektoncd/pipeline/pkg/git.Fetch\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:90\nmain.main\n\tgithub.com/tektoncd/pipeline/cmd/git-init/main.go:51\nruntime.main\n\truntime/proc.go:203"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"error","ts":1582891102.851849,"logger":"fallback-logger","caller":"git/git.go:41","msg":"Error running git [pull --recurse-submodules=yes origin]: exit status 1\nfatal: could not read Username for 'https://github.ibm.com': No such device or address\n","stacktrace":"github.com/tektoncd/pipeline/pkg/git.run\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:41\ngithub.com/tektoncd/pipeline/pkg/git.Fetch\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:93\nmain.main\n\tgithub.com/tektoncd/pipeline/cmd/git-init/main.go:51\nruntime.main\n\truntime/proc.go:203"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"warn","ts":1582891102.8520489,"logger":"fallback-logger","caller":"git/git.go:94","msg":"Failed to pull origin : exit status 1"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"error","ts":1582891102.8555176,"logger":"fallback-logger","caller":"git/git.go:41","msg":"Error running git [checkout master]: exit status 1\nerror: pathspec 'master' did not match any file(s) known to git\n","stacktrace":"github.com/tektoncd/pipeline/pkg/git.run\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:41\ngithub.com/tektoncd/pipeline/pkg/git.Fetch\n\tgithub.com/tektoncd/pipeline/pkg/git/git.go:96\nmain.main\n\tgithub.com/tektoncd/pipeline/cmd/git-init/main.go:51\nruntime.main\n\truntime/proc.go:203"}
[build-simple : git-source-simple-ghe-d55p5] {"level":"fatal","ts":1582891102.8561358,"logger":"fallback-logger","caller":"git-init/main.go:52","msg":"Error fetching git repository: exit status 1","stacktrace":"main.main\n\tgithub.com/tektoncd/pipeline/cmd/git-init/main.go:52\nruntime.main\n\truntime/proc.go:203"}


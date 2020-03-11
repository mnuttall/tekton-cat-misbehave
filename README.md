# tekton-cat-misbehave

Towards a conference talk. Work very much in progress. 

## Setup

Install Tekton 0.10.1

```sh
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.10.1/release.yaml
```

## Summary ##

Lots of workings-out and logs in this doc, so it's a stack with latest info at the top. What we have: 

- March 2nd: an 'output' resource is enough to create an auto-pvc.
- March 3rd: a 'from' clause, and file copying, are required when creating files in a workspace under a Git pipeline resource.

## March 3rd part 2. 
I have a multi-step pipeline that can git clone, and modify files. Let's assume I want to build with kaniko or buildah - next failure should be docker push. 

See march-3-docker-push/

tkn pipeline start buildah-build-pipeline -r git-source=simple-git -r docker-image=simple-docker-image
Pipelinerun started: buildah-build-pipeline-run-bxfh7
Showing logs...
[build-simple : create-dir-image-fsv4m] {"level":"info","ts":1583253261.624014,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"KO_DATA_PATH\" does not exist or is empty"}

[build-simple : git-source-simple-git-fqvd9] {"level":"info","ts":1583253261.964243,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: open /var/run/ko/refs/heads/master: no such file or directory"}
[build-simple : git-source-simple-git-fqvd9] {"level":"info","ts":1583253264.9873486,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: open /var/run/ko/refs/heads/master: no such file or directory"}
[build-simple : git-source-simple-git-fqvd9] {"level":"info","ts":1583253265.6835756,"logger":"fallback-logger","caller":"git/git.go:102","msg":"Successfully cloned https://github.com/mnuttall/simple.git @ master in path /workspace/source"}
[build-simple : git-source-simple-git-fqvd9] {"level":"warn","ts":1583253265.6836758,"logger":"fallback-logger","caller":"git/git.go:149","msg":"Unexpected error: creating symlink: symlink /tekton/home/.ssh /root/.ssh: file exists"}
[build-simple : git-source-simple-git-fqvd9] {"level":"info","ts":1583253265.7472143,"logger":"fallback-logger","caller":"git/git.go:130","msg":"Successfully initialized and updated submodules in path /workspace/source"}

[build-simple : build] {"level":"info","ts":1583253262.298409,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"KO_DATA_PATH\" does not exist or is empty"}
[build-simple : build] STEP 1: FROM golang:1.9 AS builder
[build-simple : build] Getting image source signatures
[build-simple : build] Copying blob sha256:9c35c9787a2f7a0fbd6829b8ee0df1aebe44929788186d352b3a12f2046b3948
[build-simple : build] Copying blob sha256:55cbf04beb7001d222c71bfdeae780bda19d5cb37b8dbd65ff0d3e6a0b9b74e6
[build-simple : build] Copying blob sha256:9a8ea045c9261c180a34df19cfc9bb3c3f28f29b279bf964ee801536e8244f2f
[build-simple : build] Copying blob sha256:d4eee24d4dacb41c21411e0477a741655303cdc48b18a948632c31f0f3a70bb8
[build-simple : build] Copying blob sha256:1607093a898cc241de8712e4361dcd907898fff35b945adca42db3963f3827b3
[build-simple : build] Copying blob sha256:8b376bbb244fec17e33ac56b13cc704d03f6fb06325a0cddcb88960fef7eb754
[build-simple : build] Copying blob sha256:0d4eafcc732affec32a903b8897a068e60656b4b63d052a66feb1c8fedb3ba75
[build-simple : build] Copying blob sha256:186b06a99029b895a2934a57fcb86731eace7009cc059210358cabec2e8309e2
[build-simple : build] Copying config sha256:ef89ef5c42a90ec98bda7bbef0495c1ca6f43a31d059148c368b71858de463d2
[build-simple : build] Writing manifest to image destination
[build-simple : build] Storing signatures
[build-simple : build] STEP 2: WORKDIR /go/src/github.com/mnuttall/simple
[build-simple : build] b3a3f143515124cd9143c327215cc7de825433e2b1fc8de9d192f251aa173e6c
[build-simple : build] STEP 3: COPY . .
[build-simple : build] b795a32ab1f7a0b1b43315769cf67b2301740c906cec90101993b67f37b0e5a9
[build-simple : build] STEP 4: RUN go build .
[build-simple : build] 77cd61b86a4568fbf6f411d2c9fdbbed05a1f84025bca57dc7aebb6851605681
[build-simple : build] STEP 5: RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
[build-simple : build] 671d172c2ccdb1e0c2c77cfec7fe5f1529d34757a6040e168df8d35e8a5dc4ea
[build-simple : build] STEP 6: FROM golang:1.9
[build-simple : build] STEP 7: COPY --from=builder /go/src/github.com/mnuttall/simple/app /app
[build-simple : build] 214b7f72e0b2a6b12398e14159a4705bc8b20a40c7fed8c33b4418b00303bb69
[build-simple : build] STEP 8: ENTRYPOINT ["/app"]
[build-simple : build] STEP 9: COMMIT mnuttall/simple
[build-simple : build] 262f6c1aa3637637c5c1ae52127e39c89f040023b5dbf40fbc76a4356fd998d8

[build-simple : push] {"level":"info","ts":1583253262.6077552,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"KO_DATA_PATH\" does not exist or is empty"}
[build-simple : push] Getting image source signatures
[build-simple : push] Copying blob sha256:719d45669b35360e3ca0d53d159c42ca9985eb925a6b28ac38d0ded39a1314e8
[build-simple : push] Copying blob sha256:920961b94eb367eb39736364ebd3197098b46ca7a10476b1df052ee01a6c10b3
[build-simple : push] Copying blob sha256:fa0c3f992cbd10a0569ed212414b50f1c35d97521f7e4a9e55a9abcf47ca77e2
[build-simple : push] Copying blob sha256:3b10514a95bec77489a57d6e2fbfddb7ddfdb643907470ce5de0f1b05c603706
[build-simple : push] Copying blob sha256:ce6466f43b110e66d7d3a72600262a1f1b015d9a64aad5f133f081868d4825fc
[build-simple : push] Copying blob sha256:e7dc337030bab6f268888f7b7f6b4e9031232255313eca3ec21c2625b02faeda
[build-simple : push] Copying blob sha256:f391a6f001ade91eb7f5d59cb0bc62267d9695f5547730abfa0fcf2985e89f29
[build-simple : push] Copying blob sha256:24a9d20e5beebbd886c69737cf9bed33576ba2f2f940f6055fc45424a5899cf7
[build-simple : push] Copying blob sha256:186d94bd2c6242e334cab1076bee2ca1dcdbb0ccc34921ed0474574b29c39d63
[build-simple : push] error copying layers and metadata from "containers-storage:[overlay@/var/lib/containers/storage+/var/run/containers/storage:overlay.imagestore=/var/lib/shared,overlay.mount_program=/usr/bin/fuse-overlayfs,overlay.mountopt=nodev,metacopy=on]localhost/mnuttall/simple:latest" to "docker://mnuttall/simple:latest": Error writing blob: Error initiating layer upload to /v2/mnuttall/simple/blobs/uploads/ in registry-1.docker.io: errors:
[build-simple : push] denied: requested access to the resource is denied
[build-simple : push] unauthorized: authentication required
[build-simple : push]

Fix: 
kubectl apply secrets-no-check-in/docker-secret.yaml
kubectl patch sa default --type json -p='[{"op":"add","path":"serviceaccount/secrets/-","value":{"name":"docker"}}]'

rerun pipeline:

[build-simple : push] {"level":"info","ts":1583254322.5589879,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"KO_DATA_PATH\" does not exist or is empty"}
[build-simple : push] Getting image source signatures
[build-simple : push] Copying blob sha256:920961b94eb367eb39736364ebd3197098b46ca7a10476b1df052ee01a6c10b3
[build-simple : push] Copying blob sha256:ce6466f43b110e66d7d3a72600262a1f1b015d9a64aad5f133f081868d4825fc
[build-simple : push] Copying blob sha256:fa0c3f992cbd10a0569ed212414b50f1c35d97521f7e4a9e55a9abcf47ca77e2
[build-simple : push] Copying blob sha256:3b10514a95bec77489a57d6e2fbfddb7ddfdb643907470ce5de0f1b05c603706
[build-simple : push] Copying blob sha256:719d45669b35360e3ca0d53d159c42ca9985eb925a6b28ac38d0ded39a1314e8
[build-simple : push] Copying blob sha256:e7dc337030bab6f268888f7b7f6b4e9031232255313eca3ec21c2625b02faeda
[build-simple : push] Copying blob sha256:d8a10231f165681b8e6c61c192e122a62c745d890eccf99b65d9823a724e61ca
[build-simple : push] Copying blob sha256:24a9d20e5beebbd886c69737cf9bed33576ba2f2f940f6055fc45424a5899cf7
[build-simple : push] Copying blob sha256:186d94bd2c6242e334cab1076bee2ca1dcdbb0ccc34921ed0474574b29c39d63
[build-simple : push] Copying config sha256:5b6fe0b81a35ae2cecc067ea2b822b8152f39270459c0806a414fe46b20a7c21
[build-simple : push] Writing manifest to image destination
[build-simple : push] Storing signatures

So now we can docker push! Next: I add another secret for the same docker. Equivalent to multiple git secrets. 

```
k apply -f secrets-no-check-in/docker-secret-2.yaml
kubectl patch sa default --type json -p='[{"op":"add","path":"serviceaccount/secrets/-","value":{"name":"docker2"}}]'
```

pods: 

NAME                                                            READY   STATUS       RESTARTS   AGE
buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4   0/5     Init:Error   0          119s


 ~/dev/github/tekton-cat-misbehave   master ●✚  k get pr
NAME                               SUCCEEDED   REASON   STARTTIME   COMPLETIONTIME
buildah-build-pipeline-run-sn68k   False       Failed   22s         13s
 ~/dev/github/tekton-cat-misbehave   master ●✚  k describe pr buildah-build-pipeline-run-sn68k
Name:         buildah-build-pipeline-run-sn68k
Namespace:    default
Labels:       tekton.dev/pipeline=buildah-build-pipeline
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"tekton.dev/v1alpha1","kind":"Pipeline","metadata":{"annotations":{},"name":"buildah-build-pipeline","namespace":"default"},...
API Version:  tekton.dev/v1alpha1
Kind:         PipelineRun
Metadata:
  Creation Timestamp:  2020-03-03T17:10:47Z
  Generate Name:       buildah-build-pipeline-run-
  Generation:          1
  Resource Version:    2607650
  Self Link:           /apis/tekton.dev/v1alpha1/namespaces/default/pipelineruns/buildah-build-pipeline-run-sn68k
  UID:                 a9121408-541b-42c8-ada0-5b3b6d86b377
Spec:
  Pipeline Ref:
    Name:  buildah-build-pipeline
  Pod Template:
  Resources:
    Name:  git-source
    Resource Ref:
      Name:  simple-git
    Name:    docker-image
    Resource Ref:
      Name:  simple-docker-image
Status:
  Completion Time:  2020-03-03T17:10:56Z
  Conditions:
    Last Transition Time:  2020-03-03T17:10:56Z
    Message:               TaskRun buildah-build-pipeline-run-sn68k-build-simple-j7xm7 has failed
    Reason:                Failed
    Status:                False
    Type:                  Succeeded
  Start Time:              2020-03-03T17:10:47Z
  Task Runs:
    buildah-build-pipeline-run-sn68k-build-simple-j7xm7:
      Pipeline Task Name:  build-simple
      Status:
        Completion Time:  2020-03-03T17:10:56Z
        Conditions:
          Last Transition Time:  2020-03-03T17:10:56Z
          Message:               build failed for unspecified reasons.
          Reason:                Failed
          Status:                False
          Type:                  Succeeded
        Pod Name:                buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4
        Start Time:              2020-03-03T17:10:47Z
        Steps:
          Container:  step-build
          Name:       build
          Waiting:
            Reason:   PodInitializing
          Container:  step-push
          Name:       push
          Waiting:
            Reason:   PodInitializing
          Container:  step-git-source-simple-git-5nvkb
          Name:       git-source-simple-git-5nvkb
          Waiting:
            Reason:   PodInitializing
          Container:  step-image-digest-exporter-rlmrb
          Name:       image-digest-exporter-rlmrb
          Waiting:
            Reason:   PodInitializing
          Container:  step-create-dir-image-zt8dv
          Name:       create-dir-image-zt8dv
          Waiting:
            Reason:  PodInitializing
Events:
  Type     Reason  Age   From                 Message
  ----     ------  ----  ----                 -------
  Warning  Failed  22s   pipeline-controller  TaskRun buildah-build-pipeline-run-sn68k-build-simple-j7xm7 has failed


k get pods
NAME                                                            READY   STATUS       RESTARTS   AGE
buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4   0/5     Init:Error   0          2m12s
 ~/dev/github/tekton-cat-misbehave   master ●✚  k describe pod buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4
Name:         buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4
Namespace:    default
Priority:     0
Node:         worker2.mangers.os.fyre.ibm.com/10.16.58.41
Start Time:   Tue, 03 Mar 2020 17:10:47 +0000
Labels:       app.kubernetes.io/managed-by=tekton-pipelines
              tekton.dev/pipeline=buildah-build-pipeline
              tekton.dev/pipelineRun=buildah-build-pipeline-run-sn68k
              tekton.dev/pipelineTask=build-simple
              tekton.dev/task=buildah
              tekton.dev/taskRun=buildah-build-pipeline-run-sn68k-build-simple-j7xm7
Annotations:  k8s.v1.cni.cncf.io/networks-status:
              kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"tekton.dev/v1alpha1","kind":"Task","metadata":{"annotations":{},"name":"buildah","namespace":"default"},"spec":{"inputs":{"...
              tekton.dev/release: devel
Status:       Failed
IP:           10.254.12.34
IPs:
  IP:           10.254.12.34
Controlled By:  TaskRun/buildah-build-pipeline-run-sn68k-build-simple-j7xm7
Init Containers:
  credential-initializer:
    Container ID:  cri-o://375415ad4b15f3de24cc582c6471c328084de9f09ac86ef4d73986ca218d7b8e
    Image:         gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/creds-init@sha256:959b0d9a2d43d35e15a85460cc86567d308f467ee8ec16dbd9b32f51ce75d582
    Image ID:      gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/creds-init@sha256:959b0d9a2d43d35e15a85460cc86567d308f467ee8ec16dbd9b32f51ce75d582
    Port:          <none>
    Host Port:     <none>
    Command:
      /ko-app/creds-init
    Args:
      -docker-cfg=default-dockercfg-c5xnn
      -basic-docker=docker=https://index.docker.io/v1/
      -basic-docker=docker2=https://index.docker.io/v1/
    State:          Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Tue, 03 Mar 2020 17:10:55 +0000
      Finished:     Tue, 03 Mar 2020 17:10:55 +0000
    Ready:          False
    Restart Count:  0
    Environment:
      HOME:  /tekton/home
    Mounts:
      /tekton/creds-secrets/default-dockercfg-c5xnn from tekton-internal-secret-volume-default-dockercfg-c5xnn (rw)
      /tekton/creds-secrets/docker from tekton-internal-secret-volume-docker (rw)
      /tekton/creds-secrets/docker2 from tekton-internal-secret-volume-docker2 (rw)
      /tekton/home from tekton-internal-home (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8nhf6 (ro)
      /workspace from tekton-internal-workspace (rw)

...

Events:
  Type    Reason     Age        From                                      Message
  ----    ------     ----       ----                                      -------
  Normal  Scheduled  <unknown>  default-scheduler                         Successfully assigned default/buildah-build-pipeline-run-sn68k-build-simple-j7xm7-pod-5xjp4 to worker2.mangers.os.fyre.ibm.com
  Normal  Pulled     2m11s      kubelet, worker2.mangers.os.fyre.ibm.com  Container image "gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/creds-init@sha256:959b0d9a2d43d35e15a85460cc86567d308f467ee8ec16dbd9b32f51ce75d582" already present on machine
  Normal  Created    2m11s      kubelet, worker2.mangers.os.fyre.ibm.com  Created container credential-initializer
  Normal  Started    2m11s      kubelet, worker2.mangers.os.fyre.ibm.com  Started container credential-initializer

Using tkn v0.8 with -a <==== VALUE!!!!!

tkn pipelinerun logs buildah-build-pipeline-run-r9mnm -f -n default -a
[build-simple : credential-initializer] invalid value "docker2=https://index.docker.io/v1/" for flag -basic-docker: multiple entries for url: https://index.docker.io/v1/
[build-simple : credential-initializer] Usage of /ko-app/creds-init:
[build-simple : credential-initializer]   -basic-docker value
[build-simple : credential-initializer]     	List of secret=url pairs.
[build-simple : credential-initializer]   -basic-git value
[build-simple : credential-initializer]     	List of secret=url pairs.
[build-simple : credential-initializer]   -docker-cfg string
[build-simple : credential-initializer]     	Docker .dockercfg secret file.
[build-simple : credential-initializer]   -docker-config string
[build-simple : credential-initializer]     	Docker config.json secret file.
[build-simple : credential-initializer]   -ssh-git value
[build-simple : credential-initializer]     	List of secret=url pairs.

failed to get logs for task build-simple : error in getting logs for step working-dir-initializer: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(working-dir-initializer) : container "working-dir-initializer" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step place-tools: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(place-tools) : container "place-tools" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step create-dir-image-2ndwb: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(step-create-dir-image-2ndwb) : container "step-create-dir-image-2ndwb" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step git-source-simple-git-sz6mk: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(step-git-source-simple-git-sz6mk) : container "step-git-source-simple-git-sz6mk" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step build: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(step-build) : container "step-build" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step push: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(step-push) : container "step-push" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing
failed to get logs for task build-simple : error in getting logs for step image-digest-exporter-ps7bz: error getting logs for pod buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c(step-image-digest-exporter-ps7bz) : container "step-image-digest-exporter-ps7bz" in pod "buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c" is waiting to start: PodInitializing

You can get this from kubectl logs but not with --all-containers

k logs buildah-build-pipeline-run-r9mnm-build-simple-9rjbb-pod-7dt9c -c credential-initializer
invalid value "docker2=https://index.docker.io/v1/" for flag -basic-docker: multiple entries for url: https://index.docker.io/v1/
Usage of /ko-app/creds-init:
  -basic-docker value
    	List of secret=url pairs.
  -basic-git value
    	List of secret=url pairs.
  -docker-cfg string
    	Docker .dockercfg secret file.
  -docker-config string
    	Docker config.json secret file.
  -ssh-git value
    	List of secret=url pairs.

in k get pods: 

status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-03-03T17:26:25Z"
    message: 'containers with incomplete status: [credential-initializer working-dir-initializer
      place-tools]'
    reason: ContainersNotInitialized
    status: "False"
    type: Initialized
    

-------------------------


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


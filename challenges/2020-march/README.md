# A Honking Good Time

Save the bell, save the world.

## Results

In order of who completed the challenge:

1. [Ian Coldwater], [Brad Geesaman], and [Duffie Cooley]
2. [Anthony Weems]

## Setup

The cluster is a vanilla instance of [kind] (v0.7.0) with some very strict RBAC
rules and Pod Security Policies configured for several accounts and namespaces.
You start as the "goose" in the "garden" namespace and must enumerate your
environment and discover the challenge itself. It does **NOT** require
exploiting the host in any way. It is 100% achievable through Kubernetes.


To replicate the setup execute the following from this directory:
1. `kind create cluster --config=kind-config.yaml --name honkctl`
2. `kubectl apply -f manifests/01`
3. `kubectl apply -f manifests/02`
4. `kubectl apply -f manifests/03`
5. `./goose.sh`


## Solution

The solution and greater explanation can be found in [solution.md].

Trying not to spoil the fun if folks want to give it a go :)


[kind]: https://kind.sigs.k8s.io
[solution.md]: solution.md
[Ian Coldwater]: https://twitter.com/IanColdwater
[Brad Geesaman]: https://twitter.com/bradgeesaman
[Duffie Cooley]: https://twitter.com/mauilion
[Anthony Weems]: https://twitter.com/amlweems
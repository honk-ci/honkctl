# 2019-12-20 - Holiday Beta (NA)

Purpose of this challenge was to find the challenge itself, then find a flag that contained a riddle.

Within roughly five minutes we also decided breaking the cluster would be worth "a point"

## Results

In order of cluster breakage:

1. [@bdimcheff](https://twitter.com/bdimcheff)
2. [@markyjackson5](https://twitter.com/markyjackson5)
3. [@jrrickard](https://twitter.com/jrrickard)
4. [@coldwater](https://twitter.com/iancoldwater)
5. [@johnpreese](https://twitter.com/johnpreese)
6. [@tophee](https://twitter.com/tophee)
7. [@ehashdn](https://twitter.com/ehashdn)

The answer to the riddle was: A Toy-Yoda/Toyota.

First person to tweet the answer to [@honkci](https://twitter.com/honkci) was [@coldwater](https://twitter.com/iancoldwater).

## Setup

The cluster was a vanilla instance of [kind] with some additional lightweight [RBAC rules Pod Security Policies] configured.
- Service accounts within kube-system were granted full [privileged] access to the cluster.
- [A "goose" service account was configured with namespace admin rights to the "lake" namespace.](./k8s/goose.sh)
- Both the "goose" service account and group `system:authenticated` were configured with a [PSP] that limited it to the
  3 linux capabilities required by the OCI Spec (`CAP_AUDIT_WRITE`, `KILL`, and `NET_BIND_SERVICE`) in addition to the
  usual gamut of blocking host network access, IPC etc. **NOTE** `root` was NOT restricted. This was intentional so that
  more things could be run without debugging why over twitter.
- Before starting the kubectl context used by honkctl would be switched to use the goose account, but the default admin
  admin account was not removed.


To replicate the setup execute the following from this directory:
1. `kind create cluster --name honkctl`
2. `kubectl apply -f k8s/`
3. `docker exec honkctl-control-plane sed -i 's/NodeRestriction/NodeRestriction,PodSecurityPolicy/g' /etc/kubernetes/manifests/kube-apiserver.yaml`
4. `docker restart kind-honkctl`
5. `./goose.sh`


## Notes and Observations

- Discovery of the other contexts took roughly 30min.
- Sometimes people would switch to the admin context, perform an action and then switch it back to slow other players.
- Some (looking at you [@macintoshprime](https://twitter.com/macintoshPrime)) installed OPA or tried to install other PSPs / RBAC settings to slow people down.


## Problems along the way

- The biggest problem was running out of API request quota. This forced a pivot to what I had set up as a dev tool: Slack. Many other problems occurred because of the slack pivot.
  - You cannot natively type "/honkctl" within Slack because it attempts to use that as a Slack command.
  - Usernames were not being validated or displayed properly.
  - URLs were not properly parsed and passed into the command queue.
  - Slack is not lightweight and some had trouble when the channel started moving fast with commands/responses.
- Avada Kadavra was rushed and was not validating input properly.
- Spaces in filenames were not tested against the homebrew `ls` and `cat`. Oops.

Players: If there are any other issues that occurred, please PR an update to this list as well as create an issue in this repo.



[kind]: https://kind.sigs.k8s.io
[RBAC rules Pod Security Policies]: ./k8s
[privileged]: ./k8s/priveleged.psp.yaml
[PSP]: ./k8s/default.psp.yaml

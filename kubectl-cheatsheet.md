Much of the infrastructure runs on Kubernetes (aka k8s). Here are some handy commands:

Show possibly problematic pods (i.e. non Running pods)
Look for pods that have been Pending or Terminating a long time

    kubectl get pods | grep -v Running

Get pods by label (or without label)
(see https://skia-review.googlesource.com/c/skia-public-config/+/189960 for
how to implement this on other services)

    kubectl get pods -lappgroup=perf
    kubectl get pods -l 'appgroup notin (fiddle)'

Get pods grouped by the nodes (i.e. VM) they run on

    kubectl get pods -o wide --sort-by="{.spec.nodeName}"

Get and follow the logs for a pod

    kubectl logs -f [pod]

Get a lot of info for a pod

    kubectl describe pod [pod]

"SSH into" a running pod and poke around (might need /bin/sh instead of bash)

    kubectl exec -it [pod] /bin/bash

Delete pods matching a grep e.g. "Terminat"

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod

Delete pods and you really mean it

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod --force --grace-period=0

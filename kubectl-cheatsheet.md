Cheatsheet
==========

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
    
Investigate why a pod has been rebooted (OOM or segfault are typical reasons):

    # Look for the reason of the last termination
    kubectl get pod [pod name] --output=yaml | grep -A 6 lastState
    
    # Look at the logs in the last iteration of the pod
    kubectl logs --previous [pod]

Get a lot of info for a pod

    kubectl describe pod [pod]

"SSH into" a running pod and poke around (might need /bin/sh instead of bash)

    kubectl exec -it [pod] /bin/bash

Delete pods matching a grep e.g. "Terminat"

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod

Delete pods and you really mean it

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod --force --grace-period=0

Connect a local port (e.g. 8083) to a port on the pod (e.g. 8000). [more on port-forward](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward)

    kubectl port-forward [pod] 8083:8000

Reboot a pod (first by scaling it to 0 replicas, then back to 1 [or n]).

    # scale it down, see also rs/foo for ReplicaSet and statefulset/foo for StatefulSet
    kubectl scale --replicas=0 deployment/foo
    
    # pause, maybe check on it with kubectl get pods | grep foo
    
    # bring it back.
    kubectl scale --replicas=1 deployment/foo


Setup
=====

Getting setup with kubernetes.

First install docker, by following the instructions here:

    http://go/installdocker 

Then install kubectl:

    sudo apt-get install kubectl

Connect kubectl to the skia-public project:

    gcloud container clusters get-credentials skia-public \
      --zone us-central1-a --project skia-public

You should now be able to inspect the cluster:

    kubectl get pods

Running
=======

You can run any docker image locally

    docker run -ti <imagename>

To forward a port from within the container:

    docker run -ti -p8000:8000 <imagename>

To list all recently built images:

    docker images

Activity Logs
=============

There are [logs](https://pantheon.corp.google.com/logs/viewer?project=skia-public&folder=&organizationId=433637338589&angularJsUrl=%2Flogs%2Fviewer%3Fproject%3Dskia-public%26folder%26organizationId%3D433637338589&minLogLevel=0&expandAll=false&timestamp=2019-02-07T15:46:04.340000000Z&customFacets=&limitCustomFacetWidth=true&dateRangeEnd=2019-02-07T15:46:04.591Z&interval=PT1H&resource=k8s_cluster&scrollTimestamp=2019-02-07T15:25:24.159233000Z&logName=projects%2Fskia-public%2Flogs%2Fcloudaudit.googleapis.com%252Factivity&filters=text:jcgregorio&dateRangeStart=2019-02-07T14:46:04.591Z&advancedFilter=resource.type%3D%22k8s_cluster%22%0AlogName%3D%22projects%2Fskia-public%2Flogs%2Fcloudaudit.googleapis.com%252Factivity%22%0AprotoPayload.authenticationInfo.principalEmail:@google.com) for all activity in the cluster. 

In addition the kubernetes config files are kept in git repos:

  * [skia-public](https://skia.googlesource.com/skia-public-config/)
  * [skia-corp](https://skia.googlesource.com/skia-corp-config/)

Troubleshooting
---------------

If your pod is crashing shortly after startup and you can't "SSH" into it as above, try replacing
the args in the config with

    image: # Keep this the same, or use 'debian'
    command: ["/bin/sh"]
    args: ["-c", "sleep 300"]
    
This will run your image, but not run the crashing executable, giving you a chance to poke in
and investigate.
  
Naming
======

Applications, if they contain a separator, should use "-" instead of "\_", since all names need to be valid DNS names.

skia-corp prometheus graphs
===========================

Are available at https://skia-prom.corp.goog/graph.

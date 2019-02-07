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

Get a lot of info for a pod

    kubectl describe pod [pod]

"SSH into" a running pod and poke around (might need /bin/sh instead of bash)

    kubectl exec -it [pod] /bin/bash

Delete pods matching a grep e.g. "Terminat"

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod

Delete pods and you really mean it

    kubectl get pods | grep [search] | awk '{print $1}' | xargs -L 1 kubectl delete pod --force --grace-period=0

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
  
Naming
======

Applications, if they contain a separator, should use "-" instead of "\_", since all names need to be valid DNS names.

skia-corp prometheus graphs
===========================

    gcloud container clusters get-credentials skia-corp --zone us-central1-a --project google.com:skia-corp \
    && gcloud config set project google.com:skia-corp \
    && kubectl port-forward prometheus-0 9090:9090

Then go to http://localhost:9090/ or click on the link_to_source of an alert for skia-corp.

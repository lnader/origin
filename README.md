OpenShift 3 Alpha
=================

This is the source repository for the next version of OpenShift - the third architectural revision.  It is based around [Docker](https://www.docker.io) containers and images and the [Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes) container management solution.  OpenShift adds developer  centric and organization centric workflows on top of Kubernetes, and much of the core functionality of OpenShift is designed as plugins to the core Kubernetes concepts.

Please see the [OpenShift 3 Project Enhancement Proposal (PEP)](https://github.com/openshift/openshift-pep/blob/master/openshift-pep-013-openshift-3.md) for a deeper discussion of the features you see here.

NOTE: This is a very early prototype, and as such is designed for rapid iteration around core concepts.

[![GoDoc](https://godoc.org/github.com/openshift/origin?status.png)](https://godoc.org/github.com/openshift/origin)
[![Travis](https://travis-ci.org/openshift/origin.svg?branch=master)](https://travis-ci.org/openshift/origin)

Getting Started
---------------

You'll need Docker and the Go language compilation tools installed.

1.  [Install Docker](https://docs.docker.com/installation/#installation)
    and add your user to the 'docker' group

        $ sudo usermod -G docker -a `whoami`
        $ newgrp docker

    After doing this, confirm you can list docker processes as your user:

        $ docker ps

2.  [Install the Go language toolkit](http://golang.org/doc/install) and set your [GOPATH](http://golang.org/doc/code.html#GOPATH)
3.  Clone this git repository through the Go tools:

        $ go get github.com/openshift/origin
        $ cd $GOPATH/src/github.com/openshift/origin
   
4.  Run a build

        $ go get github.com/coreos/etcd
        $ hack/build-go.sh
    
5.  Start an OpenShift all-in-one server (includes everything you need to try OpenShift)

        $ output/go/bin/openshift start
    
6.  In another terminal window, switch to the directory:

        $ cd $GOPATH/src/github.com/openshift/origin
        $ output/go/bin/openshift kube create pods -c examples/hello-openshift/hello-pod.json

7.  You should then be able to open a browser on your machine and point to [http://localhost:6061](http://localhost:6061) and see a 'Welcome to OpenShift message'.  This example is simply [running the 'openshift/hello-openshift' Docker image](https://github.com/openshift/origin/blob/master/examples/hello-openshift/hello-pod.json#L11) which is [built on Docker Hub](https://registry.hub.docker.com/u/openshift/hello-openshift/).  That [image binds to port 8080](https://github.com/openshift/origin/blob/master/examples/hello-openshift/hello_openshift.go#L16) and [prints out a simple 'Hello OpenShift' message on access](https://github.com/openshift/origin/blob/master/examples/hello-openshift/hello_openshift.go#L9).  In the Kubernetes definition, we map [that bound port in the container](https://github.com/openshift/origin/blob/master/examples/hello-openshift/hello-pod.json#L13) [to port 6061 on the host](https://github.com/openshift/origin/blob/master/examples/hello-openshift/hello-pod.json#L14) so that we can access it via the host browser.  

    You can also try the [multiple container pod](https://github.com/openshift/origin/blob/master/examples/test-pod-multi.json) example that includes a database and an admin front-end.
   
Coming soon: Vagrant environments supporting OpenShift - see [Kubernetes README.md](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/README.md) for now.

API
---

The OpenShift APIs are exposed at `http://localhost:8081/osapi/v1beta1/*`.  

* `http://localhost:8081/osapi/v1beta1/services` (placeholder)

The Kubernetes APIs are exposed at `http://localhost:8080/api/v1beta1/*`:

* `http://localhost:8080/api/v1beta1/pods`
* `http://localhost:8080/api/v1beta1/services`
* `http://localhost:8080/api/v1beta1/replicationControllers`
* `http://localhost:8080/api/v1beta1/operations`

Several experimental API objects are being prototyped upstream, and are included here for integration testing:

* `http://localhost:8080/api/v1beta1/images`
* `http://localhost:8080/api/v1beta1/imagesByRepository`
* `http://localhost:8080/api/v1beta1/imageRepositories`
* `http://localhost:8080/api/v1beta1/builds`
* `http://localhost:8080/api/v1beta1/buildConfigs`

A draft of the proposed API is available at https://rawgit.com/openshift/origin/master/api/oov3.html and is developed under the [api](./api) directory.  Expect significant changes.


FAQ
---

1. How does OpenShift relate to Kubernetes?

    OpenShift embeds Kubernetes and adds additional functionality to offer a simple, powerful, and easy-to-approach developer and operator experience for building applications in containers.  Kubernetes today is focused around composing containerized applications - OpenShift adds building images, managing them, and integrating them into deployment flows.  Our goal is to do most of that work upstream, with integration and final packaging occuring in OpenShift.  As we iterate through the next few months, you'll see this repository focus more on integration and plugins, with more and more features becoming part of Kubernetes.
    
    OpenShift tracks the Kubernetes upstream at [github.com/openshift/kubernetes](https://github.com/openshift/kubernetes).  See the wiki in that project for more on how we manage the process of integrating prototyped features.

2. What about [geard](https://github.com/openshift/geard)?

    Geard started as a prototype vehicle for the next generation of the OpenShift node - as an orchestration endpoint, to offer integration with systemd, and to prototype network abstraction, routing, SSH access to containers, and Git hosting.  Its intended goal is to provide a simple way of reliably managing containers at scale, and to offer administrators tools for easily composing those applications (gear deploy).  
    
    With the introduction of Kubernetes, the Kubelet, and the pull model it leverages from etcd, we believe we can implement the pull-orchestration model described in [orchestrating geard](https://github.com/openshift/geard/blob/master/docs/orchestrating_geard.md), especially now that we have a path to properly [limit host compromises from affecting the cluster](https://github.com/GoogleCloudPlatform/kubernetes/pull/860).  The pull-model has many advantages for end clients, not least of which that they are guaranteed to eventually converge to the correct state of the server.  We expect that the use cases the geard endpoint offered will be merged into the Kubelet for consumption by admins.
    
    systemd and Docker integration offers efficient and clean process management and secure logging aggregation with the system.  We plan on introducing those capabilities into Kubernetes over time, especially as we work with the Docker upstream to limit the impact of the Docker daemon's parent child process relationship with containers, where death of the Docker daemon terminates the containers under it
    
    Network links and their ability to simplify how software connects to other containers is planned for Docker links v2 and is a capability we believe will be important in Kubernetes as well ([see issue 494 for more details](https://github.com/GoogleCloudPlatform/kubernetes/issues/494)).  
    
    The geard deployment descriptor describes containers and their relationships and will be mapped to deployment on top of Kubernetes.  The geard commandline itself will likely be merged directly into the `openshift` command for all-in-one management of a cluster.


Contributing
------------

Contributions are welcome - a more formal process is coming soon.  In the meantime, open issues as necessary, ask questions on the OpenShift IRC channel (#openshift-dev on freenode), or get involved in the [Kubernetes project](https://github.com/GoogleCloudPlatform/kubernetes).


License
-------

OpenShift is licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/).

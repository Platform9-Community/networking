# Troubleshoot application connectivity issues


## Introduction

Debugging communication issues between one app and another in kubernetes is non trivial and needs a lot of manual steps.
This example yaml spec can be used to deploy `k8snetlook` which runs multiple checks on the host and within the problem of Pod and reports a summary of network health.

## How-to

* Change the value of key `command` under `containers` section with the required arguments to k8snetlook
* Change the value of key `kubernetes.io/hostname` under `nodeSelector` section in the yaml spec to the host on which the problem pod is running.
* Apply the yaml spec and look at the logs of `k8snetlook` pod in `k8snetlook` namespace

For more information see [k8snetlook](https://github.com/sarun87/k8snetlook/#run-within-k8s)

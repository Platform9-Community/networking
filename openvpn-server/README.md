## Helm chart for OpenVPN

The following Tutorial was tested  on PMK v 4.4 BareOS Kubernetes v1.17 and flannel CNI + MetalLB

### Prequequisites 

- Kubernetes Cluster with Dynamic PV storage class if you are testing locally please deploy the nfs-provisioning from the storage repo.

- Kubernetes with MetalLB

- Flannel CNI

This chart will install an OpenVPN server inside a kubernetes cluster. New certificates are generated on install, and a script is provided to generate client keys as needed. The chart will automatically configure dns to use kube-dns and route all network traffic to kubernetes pods and services through the vpn. By connecting to this vpn a host is effectively inside a cluster's network.

Uses
The primary purpose of this chart was to make it easy to access kubernetes services during development. It could also be used for any service that only needs to be accessed through a vpn or as a standard vpn.

Usage
helm repo add stable http://storage.googleapis.com/kubernetes-charts
helm install stable/openvpn
Wait for the external load balancer IP to become available. Check service status via: kubectl get svc

Please be aware that certificate generation is variable and may take some time (minutes). Check pod status, replacing $HELM_RELEASE with the name of your release, via:

```POD_NAME=$(kubectl get pods -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}') \
&& kubectl logs "$POD_NAME" --follow
When all components of the openvpn chart have started use the following script to generate a client key:

#!/bin/bash

if [ $# -ne 3 ]
then
  echo "Usage: $0 <CLIENT_KEY_NAME> <NAMESPACE> <HELM_RELEASE>"
  exit
fi

KEY_NAME=$1
NAMESPACE=$2
HELM_RELEASE=$3
POD_NAME=$(kubectl get pods -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_NAME=$(kubectl get svc -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_IP=$(kubectl get svc -n "$NAMESPACE" "$SERVICE_NAME" -o go-template='{{range $k, $v := (index .status.loadBalancer.ingress 0)}}{{$v}}{{end}}')
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" /etc/openvpn/setup/newClientCert.sh "$KEY_NAME" "$SERVICE_IP"
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" cat "/etc/openvpn/certs/pki/$KEY_NAME.ovpn" > "$KEY_NAME.ovpn"
```
In order to revoke certificates in later steps:
```
#!/bin/bash

if [ $# -ne 3 ]
then
  echo "Usage: $0 <CLIENT_KEY_NAME> <NAMESPACE> <HELM_RELEASE>"
  exit
fi

KEY_NAME=$1
NAMESPACE=$2
HELM_RELEASE=$3
POD_NAME=$(kubectl get pods -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" /etc/openvpn/setup/revokeClientCert.sh $KEY_NAME
```
The entire list of helper scripts can be found on templates/config-openvpn.yaml

Be sure to change KEY_NAME if generating additional keys. Import the .ovpn file into your favorite openvpn tool like tunnelblick and verify connectivity.


Certificates
New certificates are generated with each deployment, if keystoreSecret is not defined. If persistence is enabled certificate data will be persisted across pod restarts. Otherwise new client certs will be needed after each deployment or pod restart.

Certificates can be passed in secret, which name is specified in openvpn.keystoreSecret value. Create secret as follows:

kubectl create secret generic openvpn-keystore-secret --from-file=./server.key --from-file=./ca.crt --from-file=./server.crt --from-file=./dh.pem [--from-file=./crl.pem] [--from-file=./ta.key]
You can deploy temporary openvpn chart, create secret from generated certificates, and then re-deploy openvpn, providing the secret. Certificates can be found in openvpn pod in the following files:

/etc/openvpn/certs/pki/private/server.key /etc/openvpn/certs/pki/ca.crt /etc/openvpn/certs/pki/issued/server.crt /etc/openvpn/certs/pki/dh.pem

If openvpn.useCrl is set:

/etc/openvpn/certs/pki/crl.pem

And optionally (see openvpn.taKey setting):

/etc/openvpn/certs/pki/ta.key

Note: using mounted secret makes creation of new client certificates impossible inside openvpn pod, since easyrsa needs to write in certs directory, which is read-only.

For further documentation of cofigurations and issues please look at the original source of the CNF project.

https://hub.helm.sh/charts/stable/openvpn/4.2.3

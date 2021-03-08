# networking

### openvpn-server
Install an OpenVPN server inside a kubernetes cluster. New certificates are generated on install, and a script is provided to generate client keys as needed. The chart will automatically configure dns to use kube-dns and route all network traffic to kubernetes pods and services through the vpn. By connecting to this vpn a host is effectively inside a cluster's network.

### troubleshoot 
Troubleshoot application connectivity issues in your K8s cluster.

### sriov-dpdk
This is a collection of Kubernetes SRIOV/DPDK Examples yaml files the are tested to work with Platform9 Free Teir Kubernetes.

### nginx
# Ingress
Ingress is used as entry point for clients to publicly access the services on a kubernetes cluster. In a absence of an ingress the services are typically exposed to outside via a kubernetes resoure of the type loadbalancer. Loadbalancers are expensive resources in cloud and using them for every service is just not practicle even in your on-premises private cloud. Ingress controllers not only solve this problem but also provide additional features like SSL termination, host based or path based routing and the list goes on and on. Ingress controller can typically do both host based and path based routing to kubernetes services.
Ingress overcome restrictions with using kubernetes serivices like nodePort where one is restricted to use port number 30000 and above. Cloud loadbalancers also overcomes this restriction but at higher cost as the services that are exposed grow the number of load balancers that required also keep growing. Cloud load balancers are expensive items. This is where an ingress controller has an advantage as it can expose multiple services using a single cloud loadbalancer.
The cloud loadbalancer talks with ingress which in turn routes the traffic to an underlying service on kubernetes. In kubernetes both loadbalancer and ingress have service endpoints connected with each other. On public cloud ingress controller can provision cloud loadbalancer for themselves whereas on private cloud clusters use metallb as loadbalancer.

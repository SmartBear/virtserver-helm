VirtServer Helm Chart
=====================

This is a starting point for running VirtServer in kubernetes utilizing helm charts. Users might need to modify it
to their needs. This helm chart was last updated 2022-08-18 for VirtServer 3.12.1.

Managing
-------

### Installing
If your helm is setup with kubernetes correctly all you need to do to run the helm chart is the following:

`helm install virtserver . -f values.yaml`

### Upgrading
Whenever you need to update your VirtServer to a new version or simply just restart it run:

`helm upgrade virtserver . -f values.yaml`

This usually makes your VirtServer unavailable for a minute and any Virts not configured to autostart will need to be
manually started again.

### Uninstalling
When it is time to bring down VirtServer you run:

`helm uninstall virtserver`

Configuration
-------------
Within this repository there is an example of what values.yaml could look like.

### licenseServer
Where ProtectionLS license server is running. Make sure that any network configuration needed is done so that the 
VirtServer node can reach the license server. If the license server can't be reached or doesn't have a valid VirtServer
license then VirtServer won't be able to start.

**Example:**
`host.minikube.internal:1443`

**Note:** This example works for when the ProtectionLS license server is running at the same host as a minikube cluster.
Other environments will need a different domain or IP and port. Also note that ProtectionLS in this particular configuration
is not running on its default port but rather 1443.

### exposePorts
An array with exactly two elements representing the start and stop of a range of ports that should be exposed of
VirtServer hosts. These ports maps to your virtual services running in VirtServer. The port that you would access them through will
probably not be the same as the port you decide to open. If you have 10 virtual services then you probably want to expose 10 ports.
The virtual services need to run on a port within this range.

**Example:**
`[8000, 8010]`

To figure out what port it gets mapped to(and is accessible on outside the Kubernetes cluster) run:
```
$ kubectl get svc ready-api-virtserver
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                                                                AGE
ready-api-virtserver   NodePort   10.100.126.233   <none>        9090:32298/TCP,8000:31051/TCP,8001:30639/TCP,8002:32653/TCP,8003:30283/TCP,8004:30470/TCP,8005:31451/TCP,8006:31511/TCP,8007:31808/TCP,8008:31729/TCP,8009:32241/TCP,8010:30207/TCP   5d18h
```

### configStorageSize
How big storage you want to give VirtServer for storing deployments, HAR-logs, audit logs, regular logs and more.
Can start of as a small value of for example 1Gi, but ensure you then monitor storage space, so you can upgrade it or 
clean out old logs whenever needed. Valid values are anything that would be valid as [Memory resource 
units](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory).

**Example:**
`1Gi`

### image
Which VirtServer image you want to run. This allows you to upgrade versions quick and simple by just changing the version.
Need to be the full image name including repository.

**Example:**
`smartbear/ready-api-virtserver:3.12.1`

### useIngress
Set to "true" or "false" to indicate whther an ingress should be setup to let connections to virtserver be done using
an endpoint IP and paths rather than using nodeport IPs and ports. The endpoint IP is usually more stable and ports get
remapped to paths meaning your scripts can rely on it more. However when running with an ingress only REST- and 
SOAP-"virtual services" are supported.

When running with an ingress the port gets remapped, so if a virt is running on port 8004 you access it on the path 
/virt-8004.
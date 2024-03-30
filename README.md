# samba-in-kubernetes

This is a very experimental implementation of Samba AD DC for Kubernetes.

## Prerequisites

* Rook Ceph Block Storage (unless you change it)
* MetalLB (unless you change it)

## Installation

Clone this repository:
```sh
git clone https://github.com/burbuja/samba-in-kubernetes
```
Change directory:
```sh
cd samba-in-kubernetes
```
Edit the YAML file (specially both IP addresses 192.168.3.40 and 192.168.3.41):
```sh
nano addc.yaml
```
Apply it in Kubernetes:
```sh
kubectl apply -f addc.yaml
```
Scale the pods:
```sh
kubectl -n samba-ad-server scale sts/dc --replicas=2
```

## Known issues
You won't see the secondary pod IP address when you query the first one:
```sh
nslookup domain1.sink.test 192.168.3.40
nslookup domain1.sink.test 192.168.3.41
```

## License

[GNU General Public License version 3 (GPLv3)](https://github.com/burbuja/samba-in-kubernetes/blob/master/LICENSE)

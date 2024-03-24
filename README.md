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
Edit the YAML file (specially the IP Address 192.168.3.31):
```sh
nano ad-dc.yaml
```
Apply it in Kubernetes:
```sh
kubectl apply -f  ad-dc.yaml
```

## License

[GNU General Public License version 3 (GPLv3)](https://github.com/burbuja/samba-in-kubernetes/blob/master/LICENSE)

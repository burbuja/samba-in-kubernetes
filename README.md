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

Change the directory:
```sh
cd samba-in-kubernetes
```

Edit the YAML file:
```sh
nano addc.yaml
```

Apply it in Kubernetes:
```sh
kubectl apply -f addc.yaml
```

Wait until both pods are running, then press `Ctrl`+`C` to exit:
```sh
watch kubectl -n samba-ad-server get po
```

Watch the logs, then press `Ctrl`+`C` to exit:
```sh
kubectl -n samba-ad-server logs -f dc-0
kubectl -n samba-ad-server logs -f dc-1
```

## Values

These are some values that you may want to change:
* `10.233.0.10`: internal IP address (ClusterIP) for Samba DNS
* `169.254.25.10`: internal IP address for NodeLocal DNSCache
* `192.168.3.40`: external IP address for `dc-0`
* `192.168.3.41`: external IP address for `dc-1`

## Troubleshooting

You can access to each pod shell with these commands:
```sh
kubectl -n samba-ad-server exec -it dc-0 -- bash
kubectl -n samba-ad-server exec -it dc-1 -- bash
```

If you want to query the new domain, these commands may be useful in each pod:
```sh
samba-tool dns query localhost domain1.sink.test @ ALL -P
samba-tool dns query localhost _msdcs.domain1.sink.test @ ALL -P
```

You may want to run some of the following commands in each pod to delete the old IP addresses, if necessary:
```sh
EXTERNAL_IP=$(grep "$SAMBA_CONTAINER_ID" /etc/hosts | awk 'END{print $1}')
MY_DOMAIN=$(grep "$SAMBA_CONTAINER_ID" /etc/hosts | awk 'END{print $3}' | cut -f2- -d .)
sed -i -E '/^\[global]/,/^\[/{s/^(\s+)interfaces\s+=.*/\1interfaces = lo/}' /etc/samba/smb.conf
smbcontrol all reload-config
samba-tool dns delete localhost "$MY_DOMAIN" @ A "$POD_IP" -P
samba-tool dns delete localhost "$MY_DOMAIN" "$SAMBA_CONTAINER_ID" A "$POD_IP" -P
samba_dnsupdate --verbose --current-ip="$EXTERNAL_IP" --use-samba-tool --rpc-server-ip=127.0.0.1 --option=interfaces=lo
```

## License

[GNU General Public License version 3 (GPLv3)](https://github.com/burbuja/samba-in-kubernetes/blob/master/LICENSE)

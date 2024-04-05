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

Wait until the pod ```dc-0``` is running, then press Ctrl+C:
```sh
wait kubectl -n samba-ad-server get po
```

Watch the logs:
```sh
kubectl -n samba-ad-server logs -f dc-0
```

Wait until the domain is created and running, then press Ctrl+C:
```
(...)
Copyright Andrew Tridgell and the Samba Team 1992-2023
daemon 'samba' : Starting process...
Attempting to autogenerate TLS self-signed keys for https for hostname 'DC-0.domain1.sink.test'
: /usr/sbin/krb5kdc: Stash file (null) uses DEPRECATED enctype !
: /usr/sbin/krb5kdc: Stash file (null) uses DEPRECATED enctype !
: /usr/sbin/krb5kdc: krb5kdc: starting...
TLS self-signed keys generated OK
```

Scale the pods:
```sh
kubectl -n samba-ad-server scale sts/dc --replicas=2
```

Wait until the pod ```dc-1``` is running, then press Ctrl+C:
```sh
wait kubectl -n samba-ad-server get po
```

Watch the logs:
```sh
kubectl -n samba-ad-server logs -f dc-1
```

Wait until the domain is created and running, then press Ctrl+C:
```
(...)
Copyright Andrew Tridgell and the Samba Team 1992-2023
daemon 'samba' : Starting process...
Attempting to autogenerate TLS self-signed keys for https for hostname 'DC-1.domain1.sink.test'
: /usr/sbin/krb5kdc: Stash file (null) uses DEPRECATED enctype !
: /usr/sbin/krb5kdc: Stash file (null) uses DEPRECATED enctype !
: /usr/sbin/krb5kdc: krb5kdc: starting...
TLS self-signed keys generated OK
```
### Change the IP address for ```dc-0```

Get a shell to the container:
```sh
kubectl -n samba-ad-server exec dc-0 -- bash
```

Run the following commands to set the external IP address:
``` sh
sed -i -E '/^\[global]/,/^\[/{s/^(\s+)interfaces\s+=.*/\1interfaces = lo/}' /etc/samba/smb.conf
smbcontrol all reload-config
samba_dnsupdate --verbose --current-ip="$EXTERNAL_IP" --use-samba-tool --rpc-server-ip=127.0.0.1 --option=interfaces=lo
exit
```

### Change the IP address for ```dc-1```

Get a shell to the container:
```sh
kubectl -n samba-ad-server exec dc-1 -- bash
```

Run the following commands to set the external IP address:
``` sh
sed -i -E '/^\[global]/,/^\[/{s/^(\s+)interfaces\s+=.*/\1interfaces = lo/}' /etc/samba/smb.conf
smbcontrol all reload-config
samba_dnsupdate --verbose --current-ip="$EXTERNAL_IP" --use-samba-tool --rpc-server-ip=127.0.0.1 --option=interfaces=lo
exit
```

## Troubleshooting

If you want to query the new domain, these commands may be useful:
```sh
samba-tool dns query localhost domain1.sink.test @ ALL -U administrator%Passw0rd
samba-tool dns query localhost _msdcs.domain1.sink.test @ ALL -U administrator%Passw0rd
```

You may want to run some of the following commands on each container to delete the old IP addresses:
```sh
samba-tool dns delete localhost domain1.sink.test @ A OLD_IP_ADDRESS -U administrator%Passw0rd
samba-tool dns delete localhost domain1.sink.test dc-0 A OLD_IP_ADDRESS -U administrator%Passw0rd
samba-tool dns delete localhost domain1.sink.test dc-1 A OLD_IP_ADDRESS -U administrator%Passw0rd
```

## License

[GNU General Public License version 3 (GPLv3)](https://github.com/burbuja/samba-in-kubernetes/blob/master/LICENSE)

# OpenShift-CRC

Useful links:
- [CRC tool](https://github.com/crc-org/crc)
- Cloud
  - [CRC cloud tool](https://github.com/crc-org/crc-cloud)
  - [Openspot](https://github.com/ksingh7/openspot)

Download the appropriate installation package (OpenShift version) from https://github.com/crc-org/crc/releases

Scenario:
- server: backed by Windows 10 Enterprise LTSC Build 19044 / Ryzen 5 5600U 6-core, 40GB RAM, 512GB SSD NVME, Hyper-V enabled
- client: MacOS Ventura 13 with several VPN clients to take into account (...)

## Install CRC (server side)
We will install CRC [CRC v2.6.0 (OpenShift 4.10.22)](https://github.com/crc-org/crc/releases/tag/v2.6.0)

## Setup SSH key permissions properly (server side)
Once CRC is installed, let's give tighter permissions to the private key used to access the VM created by CRC: with Windows File Explorer, head to the CRC installation path `C:\Users\%USERNAME%\.crc`, after which we find the private key at `machines\crc\id_ecdsa`; Right-click on it, Properties, Security, Advanced, Permissions; remove from the list all the entries other than that of the currently logged in user, then click on OK, and again click on OK.

## Allow incoming traffic on port 6443 (server side)
To allow external clients to contact the OpenShift API on port 6443, incoming traffic must be allowed, which by default is blocked for hosts other than localhost. Of the many methods, for now we choose the least painless (and also the most insecure).

**NOTE: this step must be repeated each time we restart the VM CRC**

- Access the VM via SSH:
Open Windows Powershell and execute the following:
```
ssh -i ~/.crc/machines/crc/id_edcsa -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null core@127.0.0.1 -p 2222
```
Once inside, execute these commands
```
curl -X POST -d '{"local":"127.0.0.1:6443"}'  gateway.containers.internal/services/forwarder/unexpose
curl -X POST -d '{"local":":6443","remote":"192.168.127.2:6443"}'  gateway.containers.internal/services/forwarder/expose
```

## Setup local DNS server (client side)
Assume that client and server are connected to the same LAN and that the server has a static IP address set to `192.168.1.139`.

The steps below could be avoided by acting directly on the `/etc/hosts` file, but since we are masochists, we will use **dnsmasq**.



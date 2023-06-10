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

**Final goal**: be able to use OpenShift from the client, either via command-line (`oc`, `kubectl`) or via browser (at https://console-openshift-console.apps-crc.testing/)

Each of the sub-sections refers to either the client or the server, so note the title, e.g. "Install CRC (**server side**)".

## Install CRC (server side)
We will install CRC [CRC v2.6.0 (OpenShift 4.10.22)](https://github.com/crc-org/crc/releases/tag/v2.6.0)

After installing CRC, we have to provision the VM. Before running `crc setup`, let's configure RAM memory and monitoring. We'll avoid monitoring because, in addition to being resource-intensive, it causes problems when we go to power up the VM:

Open Windows Powershell and execute the following:
```
crc config set memory 24576
crc config set enable-cluster-monitoring false
```

Then we set up the VM
```
crc setup
```

Once created, we can decide to run crc with the default parameters, among which we also find the DNS server to be used, via the `--nameserver <address>` parameter. We should take these parameters into account in case we can no longer start the VM.

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
- Once inside, execute these commands
```
curl -X POST -d '{"local":"127.0.0.1:6443"}'  gateway.containers.internal/services/forwarder/unexpose
curl -X POST -d '{"local":":6443","remote":"192.168.127.2:6443"}'  gateway.containers.internal/services/forwarder/expose
```
- Exit the VM
```
exit
```

## Setup local DNS server (client side)
Assume that client and server are connected to the same LAN and that the server has a static IP address set to `192.168.1.139`.

The steps below could be avoided by acting directly on the `/etc/hosts` file, but since we are masochists, we will use **dnsmasq**. We'll show both approaches anyway below `:)`.

### /etc/hosts approach

The fastest method, which also requires root permissions, is as follows:

Edit the file /etc/hosts with a text editor, e.g. `vi /etc/hosts`, by adding the following lines (recall that we previously assumed `192.168.1.139` was the IP of the CRC server)

```
192.168.1.139 console-openshift-console.apps-crc.testing
192.168.1.139 oauth-openshift.apps-crc.testing
192.168.1.139 api.crc.testing
```

### dnsmasq approach

In our case, the client has many VPN clients installed. These include FortiClient VPN, which compromises the normal functioning of DNS. We will therefore follow some workarounds found online (source: [StackOverflow](https://stackoverflow.com/a/73956467/15478600));

FortiClient VPN enables the `pf` firewall + NAT DNS Redirect; We can confirm that by executing the following:

```
sudo pfctl -s nat
```

Output:

```
rdr pass inet proto udp from any to any port = 53 -> 127.0.0.1 port 53535
```

Workoaround (reset pf, **NOTE: probably has to be executed every time we restart the client**):

```
# reset the rules based off the on-disk version
sudo pfctl -N -f /etc/pf.conf

# clear the DNS cache on system
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```


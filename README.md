# ipfailover

Create ipfailover resources in OpenShift 4.x

```

### create ServiceAccount ipfailover and update clusterrole.
$ oc create sa ipfailover
$ oc adm policy add-scc-to-user privileged -z ipfailover

### create ipfailover by Deployment
$ oc create -f https://github.com/jechen0648/ipfailover/blob/main/deploy-ipfailover.yaml

### create example Appliacations that monitored by ipfailover
$ oc create -f https://raw.githubusercontent.com/lihongan/ipfailover/main/web-server-rc.yaml

### check the logs of ipfailover pods
$ oc logs ipfailover-xxxx

### check the configuration of keepalived
$ oc rsh ipfailover-xxxx
$ ps -ef
$ cat /etc/keepalived/keepalived.conf

### Reconfigure the ipfailover by updating the ENV. 
### e.g. change the virtual IP to use 192.168.10.10 and 192.168.10.11. 
$ oc set env deploy/ipfailover OPENSHIFT_HA_VIRTUAL_IPS=192.168.10.10-11

```

more Environment variables please see:
https://docs.openshift.com/container-platform/3.11/admin_guide/high_availability.html#options-environment-variables


Note: ipfailover/VRRP uses multicast by default, but multicast is not allowed in many Cloud Platforms, so please use Unicast instead for your testing.

To use multicast, ensure below rule is added to iptables INPUT chain.
```
-A INPUT -d 224.0.0.18/32 -j ACCEPT
```

### use unicast

```
### Find the worker node's IP
$ oc get node -o wide

### Specify node's IP as unicast peers
$ oc set env deploy/ipfailover OPENSHIFT_HA_UNICAST_PEERS="10.0.152.208,10.0.183.251,10.0.219.207"

### Please skip below, seems default "false" works as well
$ oc set env deploy/ipfailover OPENSHIFT_HA_USE_UNICAST="true"                                         

### capture the vrrp unicast packets on nodes.
# tcpdump -i any vrrp -nn
09:11:35.701300 IP 10.0.183.251 > 10.0.219.207: VRRPv2, Advertisement, vrid 1, prio 132, authtype simple, intvl 1s, length 20
09:11:36.708204 IP 10.0.183.251 > 10.0.219.207: VRRPv2, Advertisement, vrid 1, prio 132, authtype simple, intvl 1s, length 20
```

note: 10.0.183.251 hosts the VIP (MASTER STATE)


see also: https://sites.google.com/site/springblend/home/deploy-ipfailover-router-with-unicast-mode

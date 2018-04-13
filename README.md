# contrail-demo1
Contrail Demo: creating VM instances, virtual networks, VN policies... etc. Using OpenStack Horizon and Contrail Web UI

## 1. OpenStack CLI 

### 1.1 OpenStack Status on OpenStack node
```
openstack-status

	openstack-nova-api			active
	libvirt-bin				active
```

### 1.2. OpenStack Status on Compute node
```
openstack-status

	openstack-nova-compute			active
	libvirt-bin				active
```

## 2. OpenStack GUI / Dashboard (Horizon)

### 2.1 Login to the Dashboard - Horizon
```
http://<ip-address>/horizon	<<< Mitaka
http://<ip-address>/dashboard	<<< Ocata (since Newton?)
```

### 2.2. Create a new project (=tenant)
```
Identity > Users
"demo1"
```
Assign members (admin user with admin permissions)
```
+ admin
```

### 2.3. Create a new user (optional)
```
Identity > Users
<name> admin, netadmin 
```

### 2.4. Try to create a new VM Instance & show what needs to be done first
```
Project > Compute > Instances > Launch
```
We need a few things first:
- a Source image
- a Flavor
- a Virtual Network

... let's go ahead and create those things before we can launch VM instances ! :)

### 2.5. Create a new VM Flavor 
```
Admin > System > Flavors
"cirros256"	1 vCPU	256MB RAM	0GB	0GB
Flavor Access > + jcf
```

### 2.6. Create a new VM Image
```
Admin > System > Images
"cirros-image"	QCOW2-QEMU	cirros-0.4.0-x86_64-disk.img	public
```

### 2.7. Create a new VNetwork in OpenStack
```
Project > Other > Networking
"blue-net"	10.0.1.0/24	gw 10.0.1.1
```

### 2.8. Create 2 new VM Instances
```
Project > Compute > Instances > Launch
"blue" x2 image=cirros-image	 flavor=cirros256
```
What IP addresses assigned ?
(Assigned by Contrail via DHCP)

## 3. Contrail Virtual Network Creation

### 3.1. Check on Contrail Web UI
```
Monitor > Networking > Instances
```
On which compute nodes are the VM instances installed?
Attached to which network? 
(blue-net - yep!)

### 3.2. Login to blue1 and ping blue2 (pick VMs on different compute nodes)
```
ifconfig
route
ping 10.0.1.xxx
```
Ping should succeed

### 3.3. Create a new VNetwork in Contrail
```
Project > "demo1"	
Config > Networking > Networks
"red-net"	10.0.2.0/24	gw 10.0.2.1
```

### 3.4. Create 3 new VM Instances in OpenStack
```
Project > Compute > Instances > Launch
"red"	x3	image=cirros-image	flavor=cirros256
```

### 3.5. Ping from VM-blue to VM-red (pick VMs on different compute nodes)
```
ping 10.0.2.xxx
```
Ping should fail.
why? 
(lacking a Policy to allow traffic between VNs)

## Contrail Network Policies

### 4.1. Create VN Policy in Contrail
```
Config > Networking > Policies
"allow-ping"
Protocol=ICMP	Network=ANY <> (Bidirectional)
```

### 4.2. Attach Policy to each VN
```
Config > Networking > Networks 
Edit: blue-net
Add Network-Policy: "allow-ping"
Edit: red-net
Add Network-Policy: "allow-ping"
```

### 4.3. Ping from blue-1 to red-1
```
ping 10.0.2.xxx 
```
- Ping should succeed

CQFD! :)



## 5. Contrail Analyzer
---- TO BE CONTINUED ----

### 5.1 Create an image in OpenStack
```
Project > Compute > Images > Create Image
"Analyzer"	image=analyzer-vm-3.2.0.0-19.qcow2	format=QCOW2
```
- Image shoud become "Active"

```
Monitor > Debug > Packet Capture
Create a new Analyzer
```

### 5.2 View the Analyzer
- Show Wireshark trace
- Highlight the encapsulation used

### 5.3 Show Analytics data about the pings sent across
```
Monitor > Infrastructure > Virtual Routers > Flows
```

## 6. Contrail Service Chaining
---- TO BE CONTINUED ----

- Insert a vSRX service instance between the 2 networks...

```
Project > Compute > Images > Create Image
"vSRX-Transparent"	image=vSRX-Transparent	format=raw
"vSRX-In-Network"	image=vSRX-In-Network	format=raw
```
- Images shoud become "Active"


## 7. Contrail Gateway Router
---- TO BE CONTINUED ----

- Connect one of the VN to a physical router
    - Show the MX configuration with:
        - MP-iBGP to Contrail SDN Controller
        - Routing Instance with a specific Route Target
    - Simply add the same RT to the VN configuration

### Contrail BGP Router Configuration

Configure BGP peering in the control node:
```
Configure > Infrastructure > BGP Routers > Create BGP Router
```

| Field  | Description |
| --- | --- |
| Hostname | Enter a name for the node being added. |
| IP Address | The IP address of the node. |
| Autonomous System | Enter the AS number for the node. (BGP peer only) |
		


### Gateway router configuration (Junos OS)

Enable tunnel services:
```
chassis {
    fpc <fpc-id> {
        pic <pic-id> {
            tunnel-services {

            }
        }
    }
}
```

Configure the GRE tunnel that connects the VRF routing instance in the MX Series router to the Contrail network:
```
routing-options {
    router-id <router-id>;
    autonomous-system <ASN>; ## Must match Contrail​
    dynamic-tunnels {
        <group-name> { 
            source-address <contrail-control-node-ip-address>;
            gre;
            destination-networks {
                <subnet>; ## Specify the network on which Contrail VR peer(s) reside
            }
        }
    }
}
```

Configure BGP peering with Contrail Control node(s)
```
protocols {
    bgp {
        group contrail {
            type internal;
            local-address 10.0.1.1; ## MX Series router loopback or interface IP address
            family inet-vpn {
                any;
            }
            family route-target;    ## optional
            neighbor <contrail-control-node-ip-address>;
        }
    }
}
```


-------------------------

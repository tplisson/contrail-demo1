# contrail-demo1
Contrail Demo: creating VM instances, virtual networks, VN policies... etc. Using OpenStack Horizon and Contrail Web UI

## OpenStack GUI / Dashboard - Horizon

### 1. Login to the Dashboard - Horizon
```
http://<ip-address>/horizon	<<< 
http://<ip-address>/dashboard	<<< 
```

### 2. Create a new project (=tenant)
```
Identity > Users
"demo1"
```
Assign members (admin user with admin permissions)
```
+ admin
```

### 3. Create a new user (optional)
```
Identity > Users
<name> admin, netadmin 
```

### 4. Try to create a new VM Instance & show what needs to be done first
```
Project > Compute > Instances > Launch
```
We need a few things first:
- a Source image
- a Flavor
- a Virtual Network

... let's go ahead and create those things before we can launch VM instances ! :)

### 5. Create a new VM Flavor 
```
Admin > System > Flavors
"cirros256"	1 vCPU	256MB RAM	0GB	0GB
Flavor Access > + jcf
```

### 6. Create a new VM Image
```
Admin > System > Images
"cirros-image"	QCOW2-QEMU	cirros-0.4.0-x86_64-disk.img	public
```

### 7. Create a new VNetwork in OpenStack
```
Project > Other > Networking
"blue-net"	10.0.1.0/24	gw 10.0.1.1
```

### 8. Create 2 new VM Instances
```
Project > Compute > Instances > Launch
"blue" x2 image=cirros-image	 flavor=cirros256
```
What IP addresses assigned ?
>>> Assigned by Contrail via DHCP

## Contrail Web UI

### 9. Check on Contrail Web UI
```
Monitor > Networking > Instances
```
On which compute nodes are the VM instances installed?
Attached to which network? 
>>> blue-net - yep!

### 10. Login to blue1 and ping blue2 (pick VMs on different compute nodes)
```
ifconfig
route
ping 10.0.1.xxx
```
Ping should succeed

### 11. Create a new VNetwork in Contrail
```
Project > "demo1"	
Config > Networking > Networks
"red-net"	10.0.2.0/24	gw 10.0.2.1
```

### 12. Create 3 new VM Instances in OpenStack
```
Project > Compute > Instances > Launch
"red"	x3	image=cirros-image	flavor=cirros256
```

### 13. Ping from VM-blue to VM-red (pick VMs on different compute nodes)
```
ping 10.0.2.xxx	<<< should fail 
```
why? 
>> Policy lacking

### 14. Create VN Policy in Contrail
```
Config > Networking > Policies
"blue-red"
"red-net" <--> "blue-net" (bidirectional)
```

### 15. Attach Policy to each VN
```
Config > Networking > Networks 
Edit: blue-net
Add Network-Policy: "blue-red"
Edit: red-net
Add Network-Policy: "blue-red"
```

### 16. Ping from VM-blue-1 to VM-red-1
```
ping 10.0.2.xxx 
```
Ping should succeed.

CQFD! :)

# contrail-demo1
Contrail Demo - creating VM instances, virtual networks, VN policies...

## OpenStack GUI / Dashboard - Horizon

blabla
 
  blabla

1. Login to the Dashboard - Horizon
```
http://<ip-address>/horizon		<<< 
http://<ip-address>/dashboard	<<< 
```

2. Create a new project (=tenant)
```
Identity > Users
"demo1"
```
Assign members
```
+ admin
```

3. Create a new user
```
Identity > Users
<name> admin, netadmin 
```

4. Try to create a new VM Instance & show what needs to be done first
```
Project > Compute > Instances > Launch
```
We need a Source image
We need a Flavor
We	need a Network
... let's go ahead and create those first ! :)

4. Create a new VM Flavor 
```
Admin > System > Flavors
"cirros256"	1 vCPU 	256MB RAM		0GB	0GB
	Flavor Access > + jcf
```

5. Create a new VM Image
```
Admin > System > Images
"cirros-image"	QCOW2-QEMU cirros-0.4.0-x86_64-disk.img		public
```

5. Create a new VNetwork in OpenStack
```
Project > Other > Networking
"blue-net"	10.0.1.0/24	gw 10.0.1.1
```

6. Create 2 new VM Instances
```
Project > Compute > Instances > Launch
"blue" x2 image=cirros-image	 flavor=cirros256
```
What IP addresses assigned ?
>>> Assigned by Contrail via DHCP

7. Check on Contrail Web UI
```
Monitor > Networking > Instances
```
On which compute nodes are the VM instances installed?
Attached to which network? blue-net - yep!

7. Login to blue1 and ping blue2 (pick VMs on different compute nodes)
```
ifconfig
route
ping 10.0.1.xxx	<<< should succeed
```

8. Create a new VNetwork in Contrail
```
	Project > "jcf"	
	Config > Networking > Networks
	"red-net"		10.0.2.0/24	gw 10.0.2.1
```

9. Create 3 new VM Instances in
```
	Project > Compute > Instances > Launch
	"red" x3 image=cirros-image	 flavor=cirros256
```

10. Ping from VM-blue to VM-red (pick VMs on different compute nodes)
```
	ping 10.0.2.xxx	<<< should fail 
	why? >> Policy lacking
```

11. Create VN Policy in Contrail
```
	Config > Networking > Policies
	"blue-red"
	"red-net"	 <--> 	"blue-net" (bidirectional)
```

12. Attach Policy to each VN
```
	Config > Networking > Networks 
	Edit: blue-net
	Add Network-Policy: "blue-red"
	Edit: red-net
	Add Network-Policy: "blue-red"
```

13. Ping from VM-blue-1 to VM-red-1
```
	ping 10.0.2.xxx	<<< should succeed. 
```
CQFD! :)

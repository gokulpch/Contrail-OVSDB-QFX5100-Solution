# Contrail-OVSDB-QFX5100-Solution
BMS support using Contrail-OVSDB

## Configuration Model

The following figure depicts the configuration model used in the system.

> 
        Physical Router
              |
              | 
        Physical Interface  -----  Logical Interface
                                          |
                                          |
                                   Virtual Machine Interface  -----  Virtual Network
> 

TOR Agent receives the configuration relevant to the TOR switch. It translates the OpenContrail configuration to OVSDB and populates the relevant OVSDB table entries in the TOR switch.  The following table maps the OpenContrail configuration objects to the OVSDB tables.

> 
        Contrail Objects                OVSDB Tables

        Physical Device	              Physical Switch
        Physical Interface	          Physical Port
        Virtual Networks	          Logical Switch
        Logical Interface	          <Vlan, Physical Port> binding to Logical Switch
        L2 Unicast Route table        Unicast Remote and Local Table
        -	                          Multicast Remote Table
        -	                          Multicast Local Table
        -	                          Physical Locator Table
        -	                          Physical Locator Set Table
> 

#### TOR-Agnet Configuration

![alt text](https://github.com/gokulpch/Contrail-OVSDB-QFX5100-Solution/blob/master/png/adding_tor_agnet.png)


#### TOR-Services Node Configuration

![alt text](https://github.com/gokulpch/Contrail-OVSDB-QFX5100-Solution/blob/master/png/adding_tsn_node.png)


#### Physical Device Configuration

![alt text](https://github.com/gokulpch/Contrail-OVSDB-QFX5100-Solution/blob/master/png/adding_physical_router.png)


## Configuration on QFX-5100

* Enable OVSDB
* Set the connection protocol 
* Indicate the interfaces that will be managed via OVSDB
* Configure switch options with VTEP source (in the example below, lo0.0 is used – ensure that this address is reachable from TSN node)
* In case of pssl, update the controller details. When HA proxy is used, use the address of the HA Proxy node and use the vIP when VRRP is used between multiple nodes running HA Proxy.

> 
* set interfaces lo0 unit 0 family inet address \<router-id-reachable-on-ip-fabric\>
* set switch-options ovsdb-managed
* set switch-options vtep-source-interface lo0.0
* set protocols ovsdb passive-connection protocol tcp port \<port-number\>
* set protocols ovsdb interfaces \<interfaces-to-be-managed-by-ovsdb\>
* set protocols ovsdb controller \<tor-agent-ip\> inactivity-probe-duration 10000 protocol ssl port \<tor-agent-port\>
> 

* When using SSL to connect, CA-signed certificates have to be copied to /var/db/certs directory in the QFX. One way to get these is using the following (run on any server). 

>
* apt-get install openvswitch-common
* ovs-pki init
* ovs-pki req+sign vtep
* scp vtep-cert.pem root@tor-ip:/var/db/certs
* scp vtep-privkey.pem root@tor-ip:/var/db/certs
* cacert.pem file will be available in /var/lib/openvswitch/pki/switchca, when the above are done. This is the file to be provided in the above testbed (for ca_cert_file).
>
 
#### Caveats

It is not possible to configure both tagged and untagged logical ports on the same QFX physical port via OVSDB. So, this configuration should not be done.

#### Debug

On the QFX, the following commands show the OVSDB configuration.

* 	show ovsdb logical-switch
* 	show ovsdb interface
* 	show ovsdb mac
* 	show ovsdb controller
* 	show vlans

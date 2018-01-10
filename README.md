# ansible-cisco

Learning network automation with Ansible and GNS3.  (This is a work in progress)

## Aims

Using an Ansible playbook to:

* configure loopback interfaces on each router
* configure the backbone interfaces on each router
* configure OSPF on each router

## Environment 

* GNS3 - 2.1.0
* ansible-2.4.2.0
* python version = 2.7.12
* cisco ios 12.2(33)SRE9

### Router config

Enable password must be set

    jupiter(config)#enable secret <password>
    
A user and password must be created
    
    jupiter(config)#username john password <password>
    jupiter(config)#username john privilege 15
    
SSH must be enabled

    jupiter(config)#$crypto key generate rsa general-keys label home.test exportable ; chose 1024 bits in the modulus
    jupiter(config)#$crypto key export rsa home.test pem terminal 3des monkeymonkey

ip domain must be specified

    jupiter(config)# ip domain name home.test
    
SSHv2 must be configured

    jupiter(config)# ip ssh version 2
    

## Ansible 

/etc/ansible/hosts

    [core_routers]
    jupiter ansible_host=192.168.100.254 ansible_hostname=jupiter
    saturn ansible_host=192.168.100.253 ansible_hostname=saturn
    mars ansible_host=192.168.100.252
    neptune ansible_host=192.168.100.251
  
Key checking must be disabled  

/etc/ansible/ansible.cfg

    # uncomment this to disable SSH key host checking
    host_key_checking = False


## Errors:

Inclusion of group_vars/all.yml is fine, but group_vars/router.yml is not. Resulting error is 

    fatal: [jupiter]: FAILED! => {
       "msg": "'loopback' is undefined"
    }
    fatal: [saturn]: FAILED! => {
       "msg": "'loopback' is undefined"
    }

## Topology

This is the topology used.

![Topology](https://github.com/jmdarville/ansible-cisco/blob/master/topology.png)


The network 192.168.100.0/24 is acting as the management LAN for the control node to communicate with the routers whose FastEthernet0/0 is placed into this subnet.

The GigabitEthernet1/0 interfaces on each router will be the backbone network.


## Basic Router Configs

These are the basic configs that create a user and allow remote access through ssh. The username and passwords are published below as this is just an example using GNS3. Obviously not a production ready scenario...

###### jupiter - basic config

    enable
    conf t
    enable secret monkey
    username john password monkey
    username john privilege 15
    hostname jupiter
    line vty 0 4
    login local
    logging synchronous
    transport input telnet ssh
    crypto key generate rsa general-keys label home.test exportable
    1024
    crypto key export rsa home.test pem terminal 3des monkeymonkey
    ip ssh version 2
    ip domain name home.test
    interface FastEthernet0/0
    ip address 192.168.100.254 255.255.255.0
    duplex full
    no shut
    end

###### saturn - basic config

    enable
    conf t
    enable secret monkey
    username john password monkey
    username john privilege 15
    hostname saturn
    line vty 0 4
    login local
    logging synchronous
    transport input telnet ssh
    crypto key generate rsa general-keys label home.test exportable
    1024
    crypto key export rsa home.test pem terminal 3des monkeymonkey
    ip ssh version 2
    ip domain name home.test
    interface FastEthernet0/0
    ip address 192.168.100.253 255.255.255.0
    duplex full
    no shut
    end

###### mars - basic config

    enable
    conf t
    enable secret monkey
    username john password monkey
    username john privilege 15
    hostname mars
    line vty 0 4
    login local
    logging synchronous
    transport input telnet ssh
    crypto key generate rsa general-keys label home.test exportable
    1024
    crypto key export rsa home.test pem terminal 3des monkeymonkey
    ip ssh version 2
    ip domain name home.test
    interface FastEthernet0/0
    ip address 192.168.100.252 255.255.255.0
    duplex full
    no shut
    end

###### neptune - basic config

    enable 
    conf t
    enable secret monkey
    username john password monkey
    username john privilege 15
    hostname neptune
    line vty 0 4
    login local
    logging synchronous
    transport input telnet ssh
    crypto key generate rsa general-keys label home.test exportable
    1024
    crypto key export rsa home.test pem terminal 3des monkeymonkey
    ip ssh version 2
    ip domain name home.test
    interface FastEthernet0/0
    ip address 192.168.100.251 255.255.255.0
    duplex full
    no shut
    end


# ansible-cisco

Learning network automation with Ansible and GNS3.  (This is a work in progress)

## Contents

* [Aims](#aims)
* [Environment](#environment)
* [Ansible](#ansible)
* [Topology](#topology)
* [Basic Router Configs](#basic-router-configs)
* [Errors and Troubleshooting](#errors-and-troubleshooting)


## Aims

Using an Ansible playbook to:

* configure loopback interfaces on each router
* configure the backbone interfaces on each router
* configure OSPF on each router


## Environment 

* GNS3 - 2.1.0
* ansible-2.4.2.0
* python version = 2.7.12
* cisco ios 12.2(33)SRE9 - You need to provide your own IOS.


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

You might also need to add this to the .ssh/config file

    StrictHostKeyChecking no


## Topology

This is the topology used.

![Topology](https://github.com/jmdarville/ansible-cisco/blob/master/topology.png)


The network 192.168.100.0/24 is acting as the management LAN for the control node to communicate with the routers whose FastEthernet0/0 is placed into this subnet.

The GigabitEthernet1/0 interfaces on each router will be the backbone network.


## Basic Router Configs

These are the basic configs that create a user and allow remote access through ssh. The username and passwords are published below as this is just an example using GNS3. Obviously not a production ready scenario...

* Enable password must be set
* A user and password must be created
* SSH must be enabled
* ip domain must be specified
* SSHv2 must be configured


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
    write memory
    

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
    write memory
    

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
    write memory
    

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
    write memory
    

## Errors and Troubleshooting:

Ahh, where would we be without troubleshooting. (Answer: Back in the stone age.)

### Inclusion of group_vars/all.yml is fine, but group_vars/router.yml is not. Resulting error is 

    fatal: [jupiter]: FAILED! => {
       "msg": "'loopback' is undefined"
    }
    fatal: [saturn]: FAILED! => {
       "msg": "'loopback' is undefined"
    }
    
Apparently this is a breaking change in Ansible 2.4. This playbook should work with 2.3, but I haven't checked.
Adding a roles directory and defining the common configs in global/main.yml seemed to fix this.

But that gave way to 

### AttributeError: 'dict' object has no attribute 'rjust'
in the ios_config module

    failed: [jupiter] (item={u'ip': u'10.0.0.1', u'mask': u'255.255.255.240', u'name': u'Loopback0', u'description': u'loopback
    0'}) => {"changed": false, "item": {"description": "loopback 0", "ip": "10.0.0.1", "mask": "255.255.255.240", "name": "Loopback0"}, "module_stderr": "Traceback (most recent call last):\n  File \"/tmp/ansible_juGox5/ansible_module_ios_config.py\",
    line 532, in <module>\n    main()\n  File \"/tmp/ansible_juGox5/ansible_module_ios_config.py\", line 437, in main\n    candidate, want_banners = get_candidate(module)\n  File \"/tmp/ansible_juGox5/ansible_module_ios_config.py\", line 364, in
    get_candidate\n    candidate.add(module.params['lines'], parents=parents)\n  File \"/tmp/ansible_juGox5/ansible_modlib.zip
    /ansible/module_utils/netcfg.py\", line 405, in add\nAttributeError: 'dict' object has no attribute 'rjust'\n", "module_stdout": "", "msg": "MODULE FAILURE", "rc": 0}


This was caused by syntax (human) error on lines 11 and 12 of roles/global/tasks/mail.yml. The colons and quotation marks were causing the yaml to be misinterpreted.

    - description: "{{ item.description }}"
    - ip address: "{{ item.ip }} {{item.mask}}"

Should be

    - description {{ item.description }}
    - ip address {{ item.ip }} {{item.mask}}




### SSH Algorithm mismatch

When attempting to ssh into a router, you see the following error

    Unable to negotiate with 192.168.100.254 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
    
This is a legacy issue with OpenSSH. The diffie-hellman algorithm is not enabled by default in OpenSSH 7.0 and greater.  This page has more details. 

[OpenSSH Legacy Options](http://www.openssh.com/legacy.html)

As a workaround for this example, add the following to *.ssh/config* on the control-node. This will allow ssh access for the whole /24.

    Host 192.168.100.*
	    KexAlgorithms +diffie-hellman-group1-sha1


### Timeout trying to send command

When trying to save a running-config to startup-config, it times outs

    fatal: [mars]: FAILED! => {"changed": false, "msg": "timeout trying to send command: copy running-config startup-config", "rc": 1}
    
This is probably caused by IOS expected a prompt from the user and none is given. A nice solution is to use a handler.

[How to save IOS configurations with Ansible](https://networklore.com/how-to-save-ios_config/)


 
 

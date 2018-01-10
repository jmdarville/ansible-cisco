# ansible-cisco

Learning network automation with Ansible


## Environment 

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

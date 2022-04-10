# Ansible Multi-Credential Demo
=========

## Overview
This is a demo playbook to illustrate a couple of Ansible concepts:
1. Leveraging ansible-vault to encrypt sensitive variables and files
2. Using seperate sets of credentials for different inventory groups

Example includes: 
  - testserver1 group using ansible-vault encrypted ppassword variable
  - testserver2 group using ansible-vault encrypted private key file

**NOTE**
SSH cannot use a private key stored as a variable. It must be written to a file and given the appropriate file permisisons (0600).
Keys in this playbook are for demonstration purposes only. Please delete them and create your own as shown below.
Do not store any passwords or private keys in plain-text, encrypt sensitive information.

## Playbooks
For demonstration purposes this makes use of 3 playbooks:
1. unpack_key.yml - This playbook decrypts the private keys for use in the next playbook. 
2. multi-credential.yml - This playbook executes the tasks/main.yml file on the two groups using different sets of credentials.
3. key_cleanup.yml - This file removes the local decrypted copies of the private key file.

###### Usage example: 
where localuser is the local user account running the playbooks
Should be run as the ansible_user that the following playbook will be run as for permissions. This playbook only executes against local host without SSH.
```
ansible-playbook -i testenv.inv -k unpack_key.yml --ask-vault-pass -u localuser
ansible-playbook -i testenv.inv -k multi-credential.yml --ask-vault-pass
ansible-playbook -i testenv.inv -k key_cleanup.yml -u localuser
```
**NOTE**
In this example all sensitive data is encrypted with the same vault password but multiple vault credentials can be used. Please see documentation.
ansible-playbook will prompt for an SSH password, in this example the entered value is arbitrary as connection info is defined under group_vars.

## group_vars and inventory

Files in the group_vars folder, named after the group with or without .yml file extension for group specific variables.
The ansible_connection variable is set to local for the ansible host to run the tasks locally without needing to authenticate via SSH.

```
├── group_vars
 │   ├── testserver1
 │   └── testserver2
```

```
Inventory
[testserver1]
xx.yy.zz.123

[testserver2]
aa.bb.cc.123

[ansible]
localhost

[ansible:vars]
ansible_connection=local
```

## Ansible Vault
Vault a string as a variable named testserver1_pw
> ansible-vault encrypt_string --stdin-name ‘testserver1_pw’

Enter vault password
Enter the string and do not press enter, press ctrl+d per on screen instructions.
Output will contain the below text which can be pasted directly into a playbook
Sensitive information will not be left in the bash history

```
testserver1_pw: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30323361326464343535613533663466396230643530626163343265336231316338626534643033
          3564393335336439363862383234613338333737643662340a323438343934313639643636346635
          63373431393034323563383464656439316466333537616265613532653439663838613634303333
          3566303331613530370a386662613739303937366164353964653934636330393234333066373232
          6266
```

Vaulted variable can be referrenced as below
```
ansible_password: "{{ testserver1_pw }}”
```

###### Encrypt an existing file, named testuser2, with ansible-vault
Only leaves an encrypted file, removes unecrypted file.
> ansible-vault encrypt testuser2

###### Edit vaulted file
> ansible-vault edit testuser2

###### Decrypting options
Only leaves a decrypted file, removes the encrypted file.
> ansible-vault decrypt testuser2

Ansible copy module will decrypt the file at destination while leaving the original encrypted file intact. 
Playbooks are written restrict to localhost to unpack without distributing private to other hosts.

From documentation:
```
If you pass an encrypted file as the src argument to the copy, template, unarchive, script or assemble module, the file will not be encrypted on the target host (assuming you supply the correct vault password when you run the play).
```

## Additional Notes
This is just one solution offered. Key files or other senstive could be stored in centralized vaults such as HashiCorp Vault. Ansible vault is used to demonstrate the concept.

In this example, key files are written to the ansible users .ssh directory in their home folder. This was chosen for demonstration simplicty.
Additionally, ssh-add could be used to add or remove private keys for the local users.

# Test Environment
The test environment consists of Alpine containers chosen for their small size. Podman was used instead of docker.

Python is installed to be able to respond to Ansible.
OpenSSH installed for SSH client.

The entrypoint script generates host keys in the default path and starts SSHD.

To demonstrate that the different hosts are using different settings, the hosts are setup to use different SSH ports.

**NOTE**
In this lab example, there is some IP merge in the inventory when using both 127.0.0.1 and localhost, but the tasks executed seperately on the groups as intended. A real-world environment would not be setup in this manner.

###### Build the container image
> podman build -t alpine-test .

###### Run images with different IP:Ports
> podman run --name testserver1 -d -p 2222:22 alpine-test:latest
> podman run --name testserver2 -d -p 127.0.0.1:2322:22 alpine-test:latest

Containers can be logged into via SSH
> ssh -p 2322 testuser@0.0.0.0
> ssh -p 2322 testuser@127.0.0.1

Use ssh-keygen and ssh-copy-id to create and distribute keys.
> ssh-keygen
> ssh-copy-id -i testuser2 -p 2322 testuser@127.0.0.1

In this playbook example, the testuser2 private key file is encrypted with ansible-vault as described above and placed into the files/ directory.

License
-------

None

# Ansible init-server

Playbook site.yml will initialize a new host in order to use it later with ansible.
It manages an ansible user with ssh key connection and an admin user with ssh password


## Install
```bash
git clone https://github.com/yoanm/ansible_init-server.git
cd ansible_init-server
```

## Build
```bash
ansible-galaxy collection install -r requirements.yml 
```

## Execute
```bash
# Run in check mode + diff mode to see what will change
ANSIBLE_STDOUT_CALLBACK=yaml ansible-playbook site.yml -CD

# If ok, then really run playbook
ANSIBLE_STDOUT_CALLBACK=yaml ansible-playbook site.yml
```

ℹ️ Playbook supports tasks listing and check mode.

❗ Playbook doesn't really supports `--list-hosts` option as hosts will be filtered based on your input !
_All hosts will "wrongly" appear with that command_


## Behavior

First you have to define at least the admin username on `users_var.yml`.

List of available parameters:
 - `admin_user_name` : **Required**
    
    the username for admin user
 
 - `admin_user_shell`: 
    
    admin user shell (`/bin/bash` for instance)
 
 - `admin_user_groups`: 
    
    admin user groups list (`[sudo]` for instance)
 
 - `ansible_user_shell`: 
    
    ansible user shell (`/bin/bash` for instance)
 
 - `ansible_user_groups`: **Default: `[sudo]`** 
    
    ansible user groups list
 - `ansible_user_authorized_key_options`:

    You can enforce security by restricting ansible ssh key connection to a subnet for instance.
 
    Example: your ansible management vlan address is `10.100.100.0/24`, you can restrict ssh key to be used only on that network with `from="10.100.100.0/24"`

    Look for "ssh authorized_keys from ip" on internet for more information. 

Following parameters will be taken from host inventory and used during execution:
- `ansible_user` will be used as ansible user username
- `ansible_ssh_private_key_file` will be used as local private key (will be created if not there)
- `ansible_ssh_private_key_file` prepended with '.pub' will be used as local public key (will be created if not there)


When you execute the playbook, it will ask for following parameters:
 - The username for ssh connection (as ansible user is not there yet).

   ⚠️⚠️ must be a user with password-less sudo access.
  
   Example : "ubuntu" for newly installed ubuntu host

 - User password for ssh connection
 - The inventory hostname to initialize

Then it will :
 - Load the file `users_vars.yml` (see above)
 
 - Manage Ansible user
   - Create the ansible users with sudo access without password
     - If user doesn't exist, it will be created with a random password. And generated password will be available on `{{ username }}_at_{{ inventory_hostname }}.pass`
   - Check if either or both private or public key is missing
     - If yes, it will
       - Generate a 4096 SSH RSA key on remote host
       - Fetch public and private key to local host at the location specified by `ansible_ssh_private_key_file` host variable
       - Set `0600` permission to local files
   - Authorize ansible public ssh key **only** (others will be removed) on the remote host (optionally with key options)
   - Update SSH server to restrict ansible ssh connection to only public key (disable password)
   - Update ssh server and disable root login
   - Ensure ansible user is able to perform ssh connection from local host to remote host using ssk key
 - Manage admin user
   - Create the admin users with sudo access with password
     - If user doesn't exist, it will be created with a random password. And generated password will be available on `{{ username }}_at_{{ inventory_hostname }}.pass`
   - Ensure admin user is able to perform ssh connection from local host to remote host using password
     - Password will be asked during task execution 
       - _check `{{ username }}_at_{{ inventory_hostname }}.pass` file if user has been created_


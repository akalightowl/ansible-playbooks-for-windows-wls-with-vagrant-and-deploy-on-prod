This is repo with complete example for using ansible on Windows 10 WSL with Vagrant. 
First of all you should install WSL (Ubuntu 18.04), Vagrant for Windows and Ansible in WSL.
After that you can use deploy on vagrant box for local testing prod and deploy to VPS/VDS prod 

# Ansible

## Ansible vault

1. create vault file with passwords and put in your password as plain text

    **in WSL**

        ansible-vault create .vault_pass

    **WARNING!!!: Don't add this file to source control**

2. to solve the problem that WSL (windows bash) creates executable files and doesn't properbly handle Linux permissions, create wsl.conf within WSL (windows bash) and edit it (https://github.com/Microsoft/WSL/issues/81).

    **in WSL**

        sudo vim /etc/wsl.conf

    File content

        [automount]
        enabled = true
        options = "metadata"
        mountFsTab = false

    Restart WSL (windows bash)

3. Create file `ansible.cfg` and add path to vault_pass file

    **in WSL**

        [defaults]
        vault_password_file = ./.vault_pass

4. Create file `secrets.yml` with all secret passwords and variables for this project.

    **in WSL**

        ansible-vault create secrets.yml

    This file will be encrypted, you can edit it with vi from ansible-vault commands. Content should look like

        password: 12345678
        password2: 12345678
        ...

    You can use this content in other files with {{ password }}, {{ password2 }} and etc.

## Ansible playbook

5. Create file `vars.yml` with all vars for this project.

    *File vars.yml for example*

6. Create file `vagrant_hosts.yml` with project hosts.

    *File vagrant_hosts.yml for example*

7. Create file `vagrant_prepare.yml` with istructions for first settings server (create user, ssh and etc.)

    *File vagrant_prepare.yml for example*

9. Create file `vagrant_deploy.yml` 

    *File vagrant_deploy.yml for example*

## Vagrant

10. Create Vagrant file with your options and add forwarded port 80 to 8080

        Vagrant.configure("2") do |config|
            config.vm.box = "centos/7"
            config.vm.synced_folder ".", "/vagrant", disabled: true
            config.vm.network "forwarded_port", guest: 80, host: 8080
        end

11. Start you VM

    **in Git-Bash (or windows shell)**

        vagrant up

12. Getting SSH keys on the VMs. You need first delete `~/.ssh` in WSL, then link it with your Windows ssh key, change the persmission for your VM private key in `.vagrant/machines/default/virtualbox` to `600` from default WSL `777`.

    **Change permissions**

        chmod 0600 .vagrant/machines/default/virtualbox/<private_key_file>

    **in WSL**

        rm -R ~/.ssh
        
        ln -s /mnt/c/Users/akalightowl/.ssh ~/.ssh

        chmod 600 /mnt/c/Users/akalightowl/WORK/otlservice-ru/deploy/.vagrant/machines/default/virtualbox/private_key

        ssh -i /mnt/c/Users/akalightowl/WORK/otlservice-ru/deploy/.vagrant/machines/default/virtualbox/private_key -o PasswordAuthentication=no vagrant@127.0.0.1 -p 2222

## Deploy on vagrant. Run Angular + Express on Vagrant with WSL

13. Prepare server

    **in WSL**

        ansible-playbook -i vagrant_hosts vagrant_prepare.yml

14. Deploy

    **in WSL**

        ansible-playbook -i vagrant_hosts vagrant_deploy.yml


## Deploy on prod. Run Angular + Express on production server (VDS/VPS and etc.)

16. Create file `prod_hosts_root.yml`.

    *File prod_hosts_root.yml for example*

17. Create file `prod_hosts_user.yml`.

    *File prod_hosts_user.yml for example*

18. Create file `prod_prepare.yml` with instructions for first server settings (create user, ssh and etc.)

    *File prod_prepare.yml for example*

19. Create file `prod_deploy.yml` with instructions to deploy after adding changes to project.

    *File prod_deploy.yml for example*

20. Copy your ssh public key to prod server.

    **in WSL**

        ssh-copy-id -i ~/.ssh/id_rsa.pub root@yourServerIp

21. Create project user on production server

    **in WSL**

        ansible-playbook -i prod_hosts_root prod_create_user.yml

22. Prepare production server

    **in WSL**

        ansible-playbook -i prod_hosts_user prod_prepare.yml

23. Deploy

    **in WSL**

        ansible-playbook -i prod_hosts_user prod_deploy.yml
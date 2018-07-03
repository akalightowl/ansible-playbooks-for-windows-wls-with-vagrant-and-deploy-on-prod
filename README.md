# Ansible

## Ansuble vault

1. create vault's file with passwords and put in it your password as plain text

    **in WSL**

        ansible-vault create .vault_pass

    **WARNING!!!: Don't add this file under source control**

2. for solve problem that WSL(windows bash) create executable files and don't understand linux permission create wsl.conf from WSL(windows bash) and with linux editor (https://github.com/Microsoft/WSL/issues/81)

    **in WSL**

        sudo vim /etc/wsl.conf

    File content

        [automount]
        enabled = true
        options = "metadata"
        mountFsTab = false

    Restart WSL(windows bash)

3. Create file `ansible.cfg` and add path to vault_pass file

    **in WSL**

        [defaults]
        vault_password_file = ./.vault_pass

4. Create file `secrets.yml` with all secrets password and vars for this project.

    **in WSL**

        ansible-vault create secrets.yml

    This file be encrypted, you can edit it with vi from ansible-vault commands. Content should looks like

        password: 12345678
        password2: 12345678
        ...

    You can use this content in other files with names {{ password }}, {{ password2 }} and etc.

## Ansible playbook

5. Create file `vagrant_vars.yml` with all vars for this project.

    *File vagrant_vars.yml for example*

6. Create file `vagrant_hosts.yml` with project hosts.

    *File vagrant_hosts.yml for example*

7. Create file `vagrant_prepare.yml` with istructions for first settings server (create user, ssh and etc.)

    *File vagrant_prepare.yml for example*

8. Create file `vagrant_first_deploy.yml` with instructions for createing project and first settings

    *File vagrant_first_deploy.yml for example*

9. Create file `vagrant_deploy.yml` with instructions for deploy after adding changes to project.

    *File vagrant_deploy.yml for example*

## Vargant

10. Create Vargant file with needed for you options adn add forwarded port 80 to 8080

        Vagrant.configure("2") do |config|
            config.vm.box = "centos/7"
            config.vm.synced_folder ".", "/vagrant", disabled: true
            config.vm.network "forwarded_port", guest: 80, host: 8080
        end

11. Start you VM

    **in Git-Bash (or windows shell)**

        vargant up

12. Getting SSH keys on the VMs. You need first delete `~/.ssh` in WLS, then linked it with you Windows ssh key, change you persmission for VM private key in `.vagrant/machines/default/virtualbox` right 600 from default WLS 777.

    **in WSL**

        rm -R ~/.ssh
        
        ln -s /mnt/c/Users/akalightowl/.ssh ~/.ssh

        chmod 600 /mnt/c/Users/akalightowl/WORK/vagrants/tmp/.vagrant/machines/default/virtualbox/private_key

        ssh -i /mnt/c/Users/akalightowl/WORK/vagrants/tmp/.vagrant/machines/default/virtualbox/private_key -o PasswordAuthentication=no vagrant@127.0.0.1 -p 2222

## Deploy on vagrant. Run Angular + Express on Vagrant with WSL

13. Prepare server

    **in WSL**

        ansible-playbook -i vagrant_hosts vagrant_prepare.yml

14. First deploy with installing ng, gulp, pm2 and etc.

    **in WSL**

        ansible-playbook -i vagrant_hosts vagrant_first_deploy.yml

15. Simple deploy

    **in WSL**

        ansible-playbook -i vagrant_hosts vagrant_deploy.yml

## Deploy on prod. Run Angular + Express on production server (VDS/VPS and etc.)

16. Copy your ssh public key to prod server.

    **in WSL**

        ssh-copy-id -i ~/.ssh/id_rsa.pub root@youServerIp

17. Create file `prod_vars.yml` with all vars for this project.

    *File prod_vars.yml for example*

18. Create file `prod_hosts_root.yml`.

    *File prod_hosts_root.yml for example*

19. Create file `prod_hosts_user.yml`.

    *File prod_hosts_user.yml for example*

20. Create file `prod_prepare.yml` with istructions for first settings server (create user, ssh and etc.)

    *File prod_prepare.yml for example*

21. Create file `prod_first_deploy.yml` with instructions for createing project and first settings

    *File prod_first_deploy.yml for example*

22. Create file `prod_deploy.yml` with instructions for deploy after adding changes to project.

    *File prod_deploy.yml for example*

23. Create project user on production server

    **in WSL**

        ansible-playbook -i prod_hosts_root prod_create_user.yml

24. Login under new user and change password. Create new password in secret.yml
    It's strange point, but honestly i don't know how solve this in another way.

    **in WSL**

            ssh {{projectUser}}@{{projectIp}}
            ansible-vault edit secrets.yml

25. Prepare production server

    **in WSL**

        ansible-playbook -i prod_hosts_user prod_prepare.yml

26. First deploy

    **in WSL**

        ansible-playbook -i prod_hosts_user prod_first_deploy.yml

27. Simple deploy

    **in WSL**

        ansible-playbook -i prod_hosts_user prod_deploy.yml
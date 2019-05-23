# Pi-hole Vagrant

🥧 Prepares [Pi-hole](https://pi-hole.net) on a virtual machine (VM) via Vagrant and Ansible.

The VM will appear on your network as a local machine, providing Pi-hole's DNS services to other machines on the network, without the need for a dedicated physical box.

The VM also has its own hostname on the local network (`pihole.local`) for easy access to Pi-hole's web interface.

## Dependencies

Make sure you have the following dependencies installed beforehand:

* [VirtualBox](https://www.virtualbox.org)
* [Vagrant](https://www.vagrantup.com)
* [Ansible](https://www.ansible.com)

These should be available via your bog-standard package manager (eg: APT on Ubuntu, [Homebrew](https://brew.sh) on the Mac).

## Usage

1. Clone this repository.

2. Switch to the repository directory.

3. In the directory, run:

        vagrant up
        
4. Vagrant will prompt you to choose an interface to bridge to the local network. In general, choose the ethernet connection if the host machine is wired, but WiFi is fine to. For example:

    ```
    ==> pihole: Available bridged network interfaces:
    1) enp0s25
    2) br-5a8f6827ab14
    3) docker0
    4) br-4ec60aaeb8fe
    ==> pihole: When choosing an interface, it is usually the one that is
    ==> pihole: being used to connect to the internet.
    pihole: Which interface should the network bridge to? 1
    ```
    
5. Allow Vagrant to finish provisioning the VM. If successful, the last lines displayed will look like these:

    ```
    PLAY [Next steps] *********************

    TASK [debug] **************************
    ok: [pihole] => {
        "msg": "Pi-hole is ready for installation. Please continue with the manual installer as per the README.md file."
    }

    PLAY RECAP ****************************
    pihole                     : ok=9    changed=5    unreachable=0    failed=0
    ```
    
6. The VM should now be running, but you need to manually run the Pi-hole installer as it requires user interaction. Still within the directory of the checked out repository, run:

        $ vagrant ssh
        
    This will bring you into the virtual machine. Now switch to the root user:
    
        vagrant@pihole:~$ sudo su - root
        
    As root, switch to the installation directory and run the installer:
    
        root@pihole:~# cd Pi-hole/automated\ install/
        root@pihole:~/Pi-hole/automated install# bash install.sh
        
7. Follow the [standard installation instructions](https://github.com/pi-hole/pi-hole/#one-step-automated-install). Ensure that you choose to install the web interface.

    Once the installation is completed, you can drop out of the VM's SSH session.
    
8. Confirm access to Pi-hole's web interface by visiting `http://pihole.local` in the web browser of any machine on the network.

9. Continue to follow the setup instructions from the Pi-hole website by configuring your router to use the new DNS server.

## Troubleshooting and Tips

* Always set up a secondary DNS server in your router, so there's something to fall back on if Pi-hole is failing for some reason. E.g.: [1.1.1.1](https://1.1.1.1).

* Test whether machines on your local network are using Pi-hole for DNS lookup using the `dig` command on any external website, for example:

        dig heliomass.com
        
    The output will let you know which DNS server was used. If you don't see the one on your local network, it means the router fell back on the secondary DNS server and Pi-hole isn't working as expected.
    
* When I set this up on my own network, I found the DNS server was running on the other network interface in the VM. If you're unable to hit the DNS server, this could be the problem. Fixing it was a multi-step process:

    1. Go to `http://pihole.local` in a web browser. Sign into the site (if you don't know the password, see the next tip)
    
    2. Navigate to "Settings" on the left-hand navigation, and then "DNS" on the top navigation.
    
    3. Under "Interface listening behavior", select "Listen on all interfaces" and then scroll down click "Save" at the bottom of the page.
    
    4. Go back to the VM and run `ifconfig`, eg:
    
        ```
        $ vagrant ssh
        Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-50-generic x86_64)

         * Documentation:  https://help.ubuntu.com
         * Management:     https://landscape.canonical.com
         * Support:        https://ubuntu.com/advantage

          System information as of Wed May 22 01:53:34 UTC 2019

          System load:  0.0               Processes:             106
          Usage of /:   12.9% of 9.63GB   Users logged in:       0
          Memory usage: 23%               IP address for enp0s3: 10.0.2.15
          Swap usage:   0%                IP address for enp0s8: 10.0.1.35


         * Canonical Livepatch is available for installation.
           - Reduce system reboots and improve kernel security. Activate at:
             https://ubuntu.com/livepatch

        0 packages can be updated.
        0 updates are security updates.


        Last login: Mon May 20 16:33:32 2019 from 10.0.2.2
        vagrant@pihole:~$ ifconfig
        enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
                inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
                inet6 fe80::86:97ff:fe5e:6688  prefixlen 64  scopeid 0x20<link>
                ether 02:86:97:5e:66:88  txqueuelen 1000  (Ethernet)
                RX packets 58571  bytes 56159998 (56.1 MB)
                RX errors 0  dropped 0  overruns 0  frame 0
                TX packets 24167  bytes 2078775 (2.0 MB)
                TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

        enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
                inet 10.0.1.35  netmask 255.255.255.0  broadcast 10.0.1.255
                inet6 fe80::a00:27ff:fe56:e9b9  prefixlen 64  scopeid 0x20<link>
                ether 08:00:27:56:e9:b9  txqueuelen 1000  (Ethernet)
                RX packets 146286  bytes 33301222 (33.3 MB)
                RX errors 0  dropped 555  overruns 0  frame 0
                TX packets 32111  bytes 8248790 (8.2 MB)
                TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

        lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
                inet 127.0.0.1  netmask 255.0.0.0
                inet6 ::1  prefixlen 128  scopeid 0x10<host>
                loop  txqueuelen 1000  (Local Loopback)
                RX packets 16099  bytes 1129836 (1.1 MB)
                RX errors 0  dropped 0  overruns 0  frame 0
                TX packets 16099  bytes 1129836 (1.1 MB)
                TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        ```

    One of the interfaces will have a different IP address to the one Pi-hole tells you the DNS is running on. Use this address (but be aware that the network could reassign a different IP if the router is restarted)

* If you forget the admin password, reset from within the VM using `pihole -a -p`. For example

    ```
    $ vagrant ssh
    vagrant@pihole:~$ sudo su - root
    root@pihole:~# pihole -a -p
    Enter New Password (Blank for no password):
    ```
    
* Once the VM is set up, treat it like a regular Ubuntu VM. I.e., check packages are up to date, etc, to keep the system secure.

    You can keep Pi-hole itself up to date by periodically running `pihole -up` as the root user.
    
* You can stop the VM with `vagrant halt`, and restart it again with `vagrant up`. If you need to provision it from scratch, destroy the VM with `vagrant destroy`.

## To-Do

1. It would be nice to fully automate the install. The Pi-hole installer unofficially supports this, and Ansible can be made to prompt the user in advance for various options.
2. It would also be nice to have an Ansible playbook for updating the OS and Pi-hole.

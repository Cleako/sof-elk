# SOF-ELK VM Preparation Steps

These are the steps used to create a SOF-ELK instance as used in the FOR572 courseware.  These steps may be a good starting point for those wishing to deploy their own SOF-ELK capabilities across multiple systems via the included Ansible playbooks.

1. Virtual Machine configuration
    * Name: `FOR572 SOF-ELK`
    * 4 cores
    * 4096 MB RAM
    * Enable hypervisor applications
    * Disable 3D graphics acceleration
    * USB 3.1 Controller
    * Remove Sound
    * Remove Printer
    * Remove Camera
    * Hardware (compatibility) version so $current-1 is supported
    * 500GB SATA HDD, single file, named `FOR572 SOF-ELK.vmdk`
    * Three IDE CD/DVD-ROM devices. Disable the two that are not attached to the installer ISO.
    * Disable Side Channel Mitigations
2. CentOS Stream 9 install
    * Enable networking (DHCP)
    * Hostname: `sof-elk`
    * Timezone: `Etc/Coordinated Universal Time` (**NOT GMT**), network time enabled
    * Install URL: `https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/iso/`
    * Installation destination: ~500GB HDD, but select "I will configure partitions"
        * Click "Click here to create them automatically"
        * Change `/home` to 200GB
        * Create `/var/lib/elasticsearch` partition and leave size blank - should auto-fill disk at ~256GB
    * Click "Begin installation"
    * Set a strong root password.  It is recommended to *disable* direct use of this account after the system is installed.  This is the default action during the Ansible installation process.
    * During install, create a user
        * Full name: `SOF-ELK User`
        * Username: `elk_user`
        * Select "Make this user an administrator"
        * Set password as desired - for class, this is `forensics`.  This weak password will requires you to click "Done" twice.
    * You may need to manually remove the installation ISO file from the VMX file when initial installation is complete.
    * Reboot the VM.
3. `yum -y update` (as root or with sudo)  This may not be needed, depending on the installation method used.
4. `reboot` (as root or with sudo)
5. `yum -y install epel-release` (as root or with sudo)
6. `yum -y install git ansible` (as root or with sudo)
7. `git clone https://github.com/philhagen/sof-elk /tmp/sof-elk` (as root or with sudo)
    * Change to the desired branch in the cloned repository, e.g.`git checkout public/v20200229`.  This branch will be the same as what is deployed in the completed installation.
8. `ansible-playbook -i 127.0.0.1, --connection=local /tmp/sof-elk/ansible/sof-elk_single_vm.yml`
    * You will need a free GeoIP account to download the City and ASN databases.  [You can learn more about the GeoLite2 databases, as well as sign up for a free MaxMind account by clicking here](https://dev.maxmind.com/geoip/geoip2/geolite2/).
9. `rm -rf /tmp/sof-elk`
10. Stage evidence as required.
    1. Classroom VM (evidence not distributed publicly)
        * Lab 2.3 (Logs)
        * Lab 3.1 (NetFlow) - load with `nfdump2sof-elk.sh` script, as prescribed by the lab text.
    2. Public VM
        * Old Lab 2.3 (Logs)
        * New Lab 3.1 (NetFlow)
11. Reboot the VM.  (Technically not required, but ensures all is set up to start on boot.)

## For FOR572 Class Version Preparation

1. Run `~root/distro_prep.sh`
2. Halt and power down the VM
3. Take a release candidate snapshot
4. Reboot
5. Test:
    1. Data load (syslog, httpdlog, netflow, passivedns)
    2. `sof-elk_clear.py -i list`
6. Remove release candidate snapshot
7. Remove extra files in `*.vmwarevm` package (logs, cache, vmsd file, etc.)
8. Create zip archive for distribution

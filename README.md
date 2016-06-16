# devstack-in-virtualbox
## Setup DevStack in VirtualBox 
1. Download Virtualbox Version 5.0.20 and above: https://www.virtualbox.org/wiki/Downloads
2. Download Ubuntu image for VB: https://virtualboxes.org/images/ubuntu/
3. Boot up virtual machine in VB: http://www.wikihow.com/Install-Ubuntu-on-VirtualBox
4. Download and Install DevStack:http://docs.openstack.org/developer/devstack/

# Configure Virtual Box network
1. Open Preferences -> network
2. Add two host adaptor: vboxnet0 and vboxnet1
![Screen Shot 2016-06-16 at 2.38.55 PM.png]({{site.baseurl}}/Screen Shot 2016-06-16 at 2.38.55 PM.png)
3. Edit vboxnet0 with subnet of 192.168.122.0/24
![Screen Shot 2016-06-16 at 2.42.16 PM.png]({{site.baseurl}}/Screen Shot 2016-06-16 at 2.42.16 PM.png)
![Screen Shot 2016-06-16 at 2.42.23 PM.png]({{site.baseurl}}/Screen Shot 2016-06-16 at 2.42.23 PM.png)
4. Edit vboxnet1 the same way except the subnet is 192.168.133.0/24

Note : IF YOU ARE NOT GOING TO INSTALL FROM SCRATCH, YOU CAN DOWNLOAD THE "meadows" VDI AND CONFIGURE THE NETWORK TO USE IT.

# Boot up Ubuntu VM in virtualbox
1. Start virtualnox and click on "New"
2. Entern name "meadows", select Type as "Linux", Version as "Ubuntu (64-bit)"
	Note: I have not tried 32-bit machine, but it may work
3. Slide memory size to 2048MB
4. Check "Create a virtual hard disk now" with "VDI disk image type
5. Check "Dynamically allocated" option for hard disk
6. Double check and Create

# Configure VM network
1. Click on the VM "meadows"
2. Go to "Network" at the middle top
3. Enable adaptor1 as "NAT" on vboxnet0, which is the port to go to internet through your host using NAT
![Screen Shot 2016-06-16 at 2.53.23 PM.png]({{site.baseurl}}/Screen Shot 2016-06-16 at 2.53.23 PM.png)
4. Enable adapator2 as "Host-only Adaptor" on vboxnet1, which will get a lcoal DHCP address from the laptop and so that the devstack can work offline when the latptop is not conneted to internet
![Screen Shot 2016-06-16 at 2.55.57 PM.png]({{site.baseurl}}/Screen Shot 2016-06-16 at 2.55.57 PM.png)

# Boto up VM "meadows"
1. Power up the VM and Select the Ubuntu booting image downloaded earlier
2. Walk through the booting process and boot up the VM
3. Login to the VM at console
4. Check both NIC eth0 and eth1 are up with addresses: ifconfig
5. If not, bring up the interface and renew hdcp address: 
	$ sudo ifconfig <eth0|eth1> up
    $ sudo dhclient -v
4. On VM, ping 192.168.133.1 (through the host-only adaptor to the gateway on the laptop)
5. On VM, ping www.google.com (through the NAT adaptor to internal through the laptop)


# Install and configure devstack on Meadows
1. Make sure your laptop is connected to internet
2. Install git not not yet: 
	$ sudo apt-get install git
3. Download devstack:
	$ git clone https://github.com/openstack-dev/devstack
4. Edit "stackrc" file and add line 753 as below:
	752 HOST_IP=$(get_default_host_ip "$FIXED_RANGE" "$FLOATING_RANGE" "$HOST_IP_IFACE" "$HOST_IP" "inet")
	753 HOST_IP="192.168.133.101"
    *Note: I did this because for some reason the script cannot get the HOST_IP correctly so I hardcoded it in.
5. Install devstack
	$ ./stack.sh
6. Verify Installation by loging to Horizon on brower: https://192.168.133.1/dashboard

!!! 呆棒了， 你成功了 ！！！


# Install rejoin-stack.sh
1. Download file "rejon-stack.sh" and install to ./devstack directory
2. This file is to restart the devstack

devstack@meadows:~/devstack$ cat rejoin-stack.sh

        #!/usr/bin/env bash

        # This script rejoins an existing screen, or re-creates a
        # screen session from a previous run of stack.sh.

        TOP_DIR=`dirname $0`

        # Import common functions in case the localrc (loaded via stackrc)
        # uses them.
        source $TOP_DIR/functions

        source $TOP_DIR/stackrc

        # if screenrc exists, run screen
        if [[ -e $TOP_DIR/stack-screenrc ]]; then
            if screen -ls | egrep -q "[0-9].stack"; then
                echo "Attaching to already started screen session.."
                exec screen -r stack
            fi
            exec screen -c $TOP_DIR/stack-screenrc
        fi

        echo "Couldn't find $TOP_DIR/stack-screenrc file; have you run stack.sh yet?"
        exit 1

# References:
https://www.openstack.org/software/start/

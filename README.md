# Step-by-Step Guide to Install and Configure BastilleBSD with ZFS on FreeBSD

## 1. Ensure Your System is Up to Date
freebsd-update fetch install  
pkg update && pkg upgrade  

## 2. Install BastilleBSD
pkg install bastille 

# Alternatively, install from ports:
cd /usr/ports/sysutils/bastille 
make install clean 

## 3. Enable and Start BastilleBSD Service
sysrc bastille_enable=YES  
service bastille start 

## 4. Create a Dedicated ZFS Dataset for Bastille's Jails
# Identify your ZFS pool (assuming it's zroot):
zpool list 

# Create a ZFS dataset for Bastille jails:
zfs create zroot/bastille 

# Set the mountpoint for the dataset to /usr/local/bastille:
zfs set mountpoint=/usr/local/bastille zroot/bastille 

# Set the correct permissions on /usr/local/bastille:
chmod 0750 /usr/local/bastille 

# Verify the mountpoint and permissions:
ls -ld /usr/local/bastille 

## 5. Configure BastilleBSD to Use the ZFS Dataset
# Open the Bastille configuration file for editing (using nano):
nano /usr/local/etc/bastille/bastille.conf 

# Enable ZFS support in BastilleBSD:
bastille_zfs_enable="YES" 

# Specify the ZFS pool:
bastille_zfs_zpool="zroot" 

# No need to adjust bastille_zfs_prefix since the dataset is correctly mounted at /usr/local/bastille.

## 6. Set Up the bastille0 Interface
# Clone the lo0 interface and rename it to bastille0:
sysrc cloned_interfaces="lo1"  
sysrc ifconfig_lo1_name="bastille0"  
service netif cloneup  

# (Optional) Configure NAT and Firewall Rules for external network access:
# Add the following to /etc/pf.conf:
ext_if="em0"  # Replace "em0" with your actual external interface  

set block-policy return  
scrub in on $ext_if all fragment reassemble  
set skip on lo  

table <jails> persist  
nat on $ext_if from <jails> to any -> ($ext_if)  
rdr-anchor "rdr/*"  

block in all  
pass out quick keep state  
antispoof for $ext_if inet  
pass in inet proto tcp from any to any port ssh flags S/SA keep state  

# Enable and start the firewall:
sysrc pf_enable=YES  
service pf start  

## 7. Initialize BastilleBSD
# With BastilleBSD configured and the bastille0 interface set up, initialize BastilleBSD by bootstrapping the FreeBSD release that matches your system’s version:
bastille bootstrap 14.1-RELEASE 

## 8. Create and Manage Jails
# Check if the Jail Already Exists
# If you encounter an error that the jail already exists, you have a few options:

# 1. Start the existing jail:
bastille start myjail 

# 2. Destroy the existing jail and create a new one:
bastille stop myjail 
bastille destroy myjail 
bastille create myjail 14.1-RELEASE 192.168.1.10 

# 3. Create a jail with a different name:
bastille create anotherjail 14.1-RELEASE 192.168.1.11 

## 9. Accessing the Jail
# To access the shell of your newly created jail, use:
bastille console myjail 

## 10. Additional Configuration
# BastilleBSD offers various features and customization options. For advanced configurations, networking setups, and using templates, refer to the official BastilleBSD documentation: https://bastillebsd.org/docs

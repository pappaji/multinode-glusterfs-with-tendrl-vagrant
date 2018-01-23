# multinode-glusterfs-with-tendrl-vagrant
Multinode Glusterfs with All-in-1 Tendrl for Monitoring Vagrant 

This is instructions for how to setup a Vagrant environment with 4 VM's, i.e. a 3-node Gluster trusted storage pool (with Tendrl agents) and another node serving as my Tendrl server.  

Important Note: This is for development or demo purposes, as I’m using a much smaller hardware footprint than what is specified at https://github.com/Tendrl/documentation/wiki/Tendrl-release-v1.5.4-(install-guide).

My Test / Demo setup:

	 Physical Host HW: (1) 3.1 GHz Intel Core i7 (4 cores), 16 GB RAM, ~1 TB of disk space
	
   Vagrant/Virtual Box setup:
      4 VMs (node0..node3) - 3 node Gluster trusted storage pool, and 1 node Tendrl server on a private network (virtual)
      Each node is configured with the following: 2 vCPU, 2 GB RAM, Boot Disk 40 GB, Additional virtualHD 1 GB
		  node0 - Tendrl server (all-in-1), so etcd and graphite are co-located on node0
      node1..node3 - Gluster nodes

Setting up the Vagrant Boxes:

1. Download and install VirtualBox.

2. Download and install Vagrant.

3. Create a directory to put the scripts in, e.g. “mydir”.  Download Vagrantfile and bootstrap.sh to “mydir” directory.

4. Modify Vagrantfile if you want more than 4 nodes (VMs), e.g. node0 will be the Tendrl master, and node1..node3 are the Gluster trusted pool and Tendrl nodes (agents).  Note: a virtual hard drive will be created/allocated on each of the nodes for 1 GB capacity.

5. Modify bootstrap.sh for anything you want installed or configured on each of the nodes including adding the NTP server.

6. Create the vagrant boxes:
          vagrant up

7. If “vagrant up” ran successfully, you would now see node0..node3 running.

8. To log into each node, perform the following:
          vagrant ssh <VM_name>
          Password for vagrant and root is “vagrant”
          
9. Ensure passwordless SSH is setup on node1..node3 (in my example I am using root).

	E.g. Run “ssh-keygen” as root on each of the nodes
	Copy /root/.ssh/id_rsa.pub (from the Tendrl master or source) into the /root/.ssh/authorized_keys file on each the Tendrl nodes.
	Update /etc/sshd_config on each of the nodes accordingly - ensure the following are not commented:
          PermitRootLogin yes
          RSAAuthentication yes
          PubkeyAuthentication yes
          PasswordAuthentication yes
          
	"service ssh restart" or reboot VMs for changes to take effect.
  
10. Update /etc/hosts on each of the nodes so they can talk to each other.

11. Verify you can ssh (without password) from node0 to node1..node3.  This will create entries in /root/.ssh/known_hosts if you’re successful.

12. You’ll need to partition the disks and get those mounted on each of the 3 nodes (node1..node3), as well as create the XFS filesystem.

	E.g. fdisk /dev/sdb
		n 	<— new partition
		p 	<— primary partition type
		<press enter for all the defaults till partition completed
		w	<— writes the partition table
	mkfs.xfs /dev/sdb1	<— create XFS filesystem
	parted /dev/sdb print	<— verifies XFS created

	If you left the bootstrap.sh instant, all you need to do is uncomment the “# /dev/sdb1 …” entry in /etc/fstab and do a “mount -a” to mount the brick

	Use “df -k” to verify the bricks is mounted

13. Setup the Gluster Trusted Storage Pool, configure bricks, and create and start volume from node1.  Follow instructions mentioned in https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart.

E.g.
	gluster peer probe node2
  gluster peer probe node3

  gluster peer status	<— verify Gluster trusted storage pool established

  gluster volume create vol1 replica 3 node1:/bricks/brick1 node2:/bricks/brick1 node3:/bricks/brick1
  gluster volume start vol1

  gstatus -a	<— verify cluster and volumes are healthy if you installed it as part of bootstrap.sh

14. You’re now ready to deploy Tendrl using tendrl-ansible.  
   Go to https://github.com/Tendrl/documentation/wiki/Tendrl-Releases to find the latest Tendrl release installation instructions, e.g. /usr/share/doc/tendrl-ansible-1.5.4/README.md.


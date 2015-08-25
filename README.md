#Isilon Operations Lab

## Overview

In this lab we are going to setup a non secure cluster with Isilon 7.2.0.4 and HDP 2.3 with Ambari 2.1.  

[Note: Ambari 2.1/HDP 2.3 with Isilon 7.2.0.4 is a preview feature only. The isilon patch shared in this lab is only for lab exercise]

*In this lab you will install 2 VMs
*Isilon Simulator 7.2.0.3
*Then apply the patch for 7.2.0.4. [This allows for installation of Ambari 2.1]
*HDP 2.3 with Ambari 2.1 on CentOS 6.7

## Assumption

This lab assumes that you have VMware Fusion installed on your laptop. If you do not have VMWare Fusion software, please download a 30 day trial version of the software.

## Isilon Simulator

Following steps are for setting up the simulator :-

### Step 1

Download the simulator and follow the install guide for Simulator to set it up. This sets up the isilon with System zone.
	
Please read and follow the instructions to setup the simulator.
It is important that you follow the instructions to the dot to create the networking correctly. 

	
Isilon 7.2.0.3 Simulator - https://www.dropbox.com/s/pdfk57upckpofdg/EMC_Isilon_OneFS_7.2.0.3_Simulator.zip?dl=0
Simulator install and setup guide - https://www.dropbox.com/s/0pw6gv9prplux4e/Isilon_OneFS_Simulator_Guide.pdf?dl=0

### Step 2
	Apply the 7.2.0.4 patch

###Download the patch from https://www.dropbox.com/s/d377sftchwpc6i6/patch-hdfs-7-2-0-3-to-7-2-0-4-preview-1.tgz?dl=0

Instructions to  apply the patch
		Follow the README file to apply the patch - https://www.dropbox.com/s/h3pdmye6cp69asq/README%20%281%29?dl=0

### Step 3

Install the 30 day temporary key for Isilon Simulator
Login to isilon simulator VM as a root user

    isi activate license ILLEG-ALC2V-GPS5T-RSMGT-5XMGD


### Setting up a zone for HDFS

Create a Zone

*Decide on a Zone Name. Ensure that the new zone that you want to create does not exist.
*For the purpose of example we will call the zone “zonehdp”. You can name it to your organization’s liking. Replace it with the version name that you want to assign.
 
    hwxisi1-1# isi zone zones list
 
/ifs is the default share across the nodes. Create a new directory for your zone under a directory “isitest”. 
isitest is just another hierarchy for the documentation purpose.
 
    hwxisi1-1# mkdir -p /ifs/isitest/zonehdp
      
Create the zone
 
    hwxisi1-1# isi zone zones create --name zonehdp --path /ifs/isitest/zonehdp
  
Attach a pool of ip addresses to the zone - Associate an IP address pool with the zone. In this step you are creating the pool. Get the pool from your Isilon Admin.  In this step replace the pool name, ip address range and zonename to an appropriate value.

    hwxisi1-1# isi networks create pool --name subnet0:poolhdp --ranges 172.18.150.110-172.18.150.119 --access-zone zonehdp --access-zone zonehdp --ifaces=1:ext-1


Create the HDFS root directory. This is usually called hadoop and must be within the access zone directory.
 
Set the HDFS root directory for the access zone
 
Create an indicator file so that we can easily determine when we are looking your Isilon cluster via HDFS.
 
 
hwxisi1-1# mkdir -p /ifs/isitest/zonehdp/hadoop
 
hwxisi1-1# isi zone zones modify zonehdp --hdfs-root-directory /ifs/isitest/zonehdp/hadoop;
 
hwxisi1-1# touch /ifs/isitest/zonehdp/hadoop/THIS_IS_ISILON_isitest_zonehdp
 
 Check the hdfs thread settings and Block Size. If it is not set, set it using the isilon documentation in the appendix. . This is a one time activity
 
 
hwxisi1-1# isi hdfs settings modify --server-threads 256
hwxisi1-1# isi hdfs settings modify --default-block-size 128M


  Create the users and directories  
  
  Download the following script and execute it by passing the zonename - https://www.dropbox.com/s/ff0eaivlef8947m/isilon_create_user.sh?dl=0
  hwxisi1-1# chmod + x isilon_create_user.sh
  hwxisi1-1# ./isilon_create_user.sh <zonename>

Extract the Isilon Hadoop Tools to your Isilon cluster. This can be placed in any directory under /ifs. It is recommended to use /ifs/isiloncluster1/scripts where isiloncluster1 is the name of your Isilon cluster·  
 	 
    hwxisi1-1# bash /ifs/isitest/scripts/isilon-hadoop-tools/onefs/isilon_create_directories.sh --dist hwx --fixperm --zone zonehdp
  
Execute the following additonal steps for a temporary bug :-

isi zone zones view zonehdp

Get the ZoneID from the following
 
isi zone zones view zonehdp
 
Replace the zoneid in the following command and execute it.
 
isi_run -z <zoneid>  "chown -R hdfs /ifs/isitest/zonehdp/hadoop
 


Restart Services
The command below will restart the HDFS service on Isilon to ensure that any cached user mapping rules are flushed. This will temporarily interrupt any HDFS connections coming from other Hadoop clusters
 
hwxisi1-1# isi services isi_hdfs_d disable ; isi services isi_hdfs_d enable
 



HDP CentOS Node
OS setup

Setup the CentOS VM.

Download the CentOS 6.5 Iso
https://www.dropbox.com/s/njx0zwmic7og6db/CentOS-6.7-x86_64-bin-DVD1.iso?dl=0

Use the iso to create an new VM
Assign an admin user and password. You should be able to use the same password for your root user

Next Steps use the following Scripts to create single node VM. Once you have the centos up and running, get the IP address of the machine.

Installation Steps

ssh into the machine as root. Get the ipaddress of the machine
	$ ifconfig

      b. Assign a hostname to your machine (I will use hdpdemo.hortonworks.com)
	$ hostname hdpdemo.hortonworks.com
	$ vi /etc/hosts
		Map the ipaddress in step a to the hostname
	$ vi /etc/security/sysconfig

    c. Download the following scripts. This helps in automated deployment of Ambari and its agents.
	wget https://www.dropbox.com/s/s91lintb4xhrqic/ambariInstall.sh?dl=0 -O ambariInstall.sh

$ chmod +x ambariInstall.sh

   d.  Run the scripts
	$ ./ambariInstall.sh

 The above scripts installs Ambari Server and the agent and starts up the ambari server. If you do not want to run the above steps you can do it manually by following the documentation from http://docs.hortonworks.com.

The next steps is use the Ambari Install to install HDP.

1.	Browse to http://<ambari-host>:8080/.
2.	Login using the following account:
Username: admin
Password: admin
 
Deploy a Hortonworks Hadoop Cluster with Isilon for HDFS
You will deploy Hortonworks HDP Hadoop using the standard process defined by Hortonworks. Ambari Server allows for the immediate usage of an Isilon cluster for all HDFS services (NameNode and DataNode), no reconfiguration will be necessary once the HDP install is completed.
1.	Configure the Ambari Agent on Isilon.
isiloncluster1-1# isi zone zones modify zonehdp --hdfs-ambari-namenode \ <smartconnectip/ip from ip pool>
 isiloncluster1-1# isi zone zones modify zonehdp --hdfs-ambari-server <hostname/ip of the ambari server>
2.	Login to Ambari Server.
3.	Welcome: Specify the name of your cluster hdpdemo.
4.	Select Stack: Select the HDP 2.3 stack.
5.	Install Options:
Specify your Linux hosts that will run HDP for your HDP cluster installation in the Target Hosts text box and the Isilon Zone Name Node IP Address.
Choose Perform Manual install
Click Next. You may see a warnings. Ignore them.
6.        Choose Services: 
Select all the services. There are more services in HDP 2.3 than in this screenshot



6. Assign Masters:

Assign NameNode and SNameNode components to the Isilon. 
Assign the rest of the nodes to HDP components

7. Assign Slaves and Clients: 

Assign the DataNode to Isilon host (No Client)
Rest of the components are assigned to the Compute node (HDP Sandbox)

8. Customize the services
Change the port for webhdfs to 8082 under HDFS
dfs.namenode.http-address = <hostname>:8082
Enter the password for all the required services

1.	Review: Carefully review your configuration and then click Deploy.
 
2.	After a successful installation, Ambari will start and test all of the selected services.  Sometime it may fail for the first time around. You may need to retry couple of times. Review the Install, Start and Test page for any warnings or errors. It is recommended to correct any warnings or errors before continuing.



Validation

Login to Ambari using admin/admin
Under MapReduce, run the “Service Check”


This will ensure that the service is up and running.
	


	





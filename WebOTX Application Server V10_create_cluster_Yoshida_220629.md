# WebOTX Application Server V10 Cluster Construction Guide  (EXPRESSCLUSTER X)

## Preface
This document describes the procedure in order to create a 2 nodes active-standby cluster of WebOTX Application Server 10.x (WebOTX AS) by EXPRESSCLUSTER X 4 on Windows.

## System configuration
### Applicable software
### OS
- Windows Server 2012/2012 R2/2016

### Software
- WebOTX Application Server V10.x
- EXPRESSCLUSTER X 4 for Windows
- Java SE 8/11

## Setup procedure
### Configuring a cluster system
Install EXPRESSCLUSTER X and WebOTX AS on each node by following the procedures in the product manuals.
#### [Reference product manual]
- EXPRESSCLUSTER X Installation & Configuration Guide
    - Chapter 4 Installing EXPRESSCLUSTER
- WebOTX Application Server Setup Guide

After installing EXPRESSCLUSTER X 4.x and WebOTX AS, create a cluster by following the procedures.

### Setting properties file for domain creation
This section describes a sample of the properties file for domain creation (domain name.properties). When creating a domain, note the following:

1.	Set the different port number for each domain to avoid port confliction.
2.	Make sure that the port numbers used are not the same, including the domain you create separately.
3.	Change the ID and password for managing the domain as necessary.

The domain-name.properties (domain1.properties by default) is created in <INSTALL_ROOT> during installation. If you create multiple domains, copy the file for domain creation. Note the above three points, create the domain creation file, and then create the domains.

- domain1.properties
<pre>
domain.hostname = <b>localhost</b>
domain.name = <b>domain1</b>						        <- Set the name of the domain
domain.admin.user = <b>admin</b> 				                <- Set the domain admin user name
domain.admin.password = <b>adminadmin</b> 			         	<- Set the password for the domain admin user
domain.admin.port = <b>6212</b>
domain.admin.jmxmp.port = <b>6712</b>
domain.http.port = <b>80</b>
domain.https.port = <b>443</b>
domain.http.admin.port = <b>5858</b>
domain.http.adminrest.port = <b>20101</b>
domain.http.ajp.port = <b>8099</b>
domain.jms.port = <b>9700</b>
domain.jms.user.port = <b>9701</b>
domain.jms.admin.port = <b>9702</b>
domain.java.debugger.port = <b>9010</b>
domain.ipv6-enable = <b>false</b>
domain.embedded-iiop-service.port = <b>7780</b>

#ObjectBroker Service Configs
server.corba-service.oadj.Port = <b>9826</b>
server.corba-service.namesv.NameServicePort = <b>2809</b>
server.corba-service.namesv.NameServiceRoundRobin = true
server.corba-service.oad.OadPort = <b>9825</b>
 
### TPMonitorManagerService Setup Properties (Standard / Enterprise Edition only) ###
tpsystem.IIOPListener.listenerPortNumber = <b>5151</b>
tpsystem.AJPListener.listenerPortNumber = <b>20102</b>
</pre>

- domain2.properties
<pre>
domain.hostname = <b>localhost</b>
domain.name = <b>domain2</b> 					        	<- Set the name of the domain
domain.admin.user = <b>admin</b> 					        <- Set the domain admin user name
domain.admin.password = <b>adminadmin</b>					<- Set the password for the domain admin user
domain.admin.port = <b>16212</b>
domain.admin.jmxmp.port = <b>16712</b>
domain.http.port = <b>8081</b>
domain.https.port = <b>8443</b>
domain.http.admin.port = <b>15858</b>
domain.http.adminrest.port = <b>30101</b>
domain.http.ajp.port = <b>18099</b>
domain.jms.port = <b>19700</b>
domain.jms.user.port = <b>19701</b>
domain.jms.admin.port = <b>19702</b>
domain.java.debugger.port = <b>19010</b>
domain.ipv6-enable = <b>false</b>
domain.embedded-iiop-service.port = <b>17780</b>
 …
#ObjectBroker Service Configs
server.corba-service.oadj.Port = <b>19826</b>
server.corba-service.namesv.NameServicePort = <b>12809</b>
server.corba-service.namesv.NameServiceRoundRobin = true
server.corba-service.oad.OadPort = <b>19825</b>
 …
### TPMonitorManagerService Setup Properties (Standard / Enterprise Edition only) ###
tpsystem.IIOPListener.listenerPortNumber = <b>15151</b>
tpsystem.AJPListener.listenerPortNumber = <b>30102</b>
</pre>

### Setting cluster environment construction on Windows (A 2node active-standby cluster environment)
This section describes the procedure for constructing a 2 node active-standby cluster environment.  
In this procedure, the active server is defined as N1 node and the standby server as N2 node.
```
+----------------------------------------------+
| Failover group name         |	 webotx1       |
+----------------------------------------------+
| Floating IP address         | 192.168.1.111  |
+----------------------------------------------+
| Virtual host nam            | webotx1        |
+----------------------------------------------+
| Partition for disk resource | Z:             |
+----------------------------------------------+
| JNDI server name            | aps1jndi       |
+----------------------------------------------+
            Table1. sample values
```

### Setting EXPRESSCLUSTER
Refer to the EXPRESSCLUSTER manual to set up a 2 node active-standby cluster environment. 
#### [Reference product manual]
- EXPRESSCLUSTER X Installation & Configuration Guide       
    - Chapter 6　Creating the cluster configuration data

Configure a failover group with the following resources for the cluster, upload the configuration to the cluster and start the cluster with Cluster WebUI.   
This guide assumes a shared disk configuration, but you can configure a mirror disk as well.
```
+-----------------------------------------------------------------------+
|Failover group                                                         |
+-------------------------------+-------------------+-------------------+
|Floating IP resource           | Resource name　   | fip1              |
|				+-------------------+-------------------+
|                               | IP address 	    | 192.168.1.111     |
+-------------------------------+-------------------+-------------------+
|Virtual computer name resource | Resource name     | vcom1             |
|				+-------------------+-------------------+
|	        		| Virtual host name | webotx1           |
+-------------------------------+-------------------+-------------------+
|Disk resource	                | Resource name     | sd1               |
|				+-------------------+-------------------+
|	                        | drive             | Z:                |
+-------------------------------+-------------------+-------------------+
|Script resource                | Resource name     | script1           |
|				+-------------------+-------------------+
|                               | script	    | start.bat,stop.bat|
+-----------------------------------------------------------------------+
                            Table2. sample values
```


### Creating WebOTX AS domain
1. Delete WebOTX AS domain [N1, N2]  
After installing WebOTX AS and creating the environment, delete the WebOTX AS domain once to build the cluster environment.

    (I) Stopping domain 
    
    Go to <INSTALL_ROOT> on the command prompt and check the current domain startup status.
        
        .¥bin¥otxadmin list-domains
    If the domain is running, stop it with the following command.
        
        .¥bin¥otxadmin stop-domain "domain name"

    (II) Stopping the WebOTX AS Agent service
    
    If the WebOTX AS Agent service is running, stop it using one of the following methods.
    - Method A: stopping the service using Windows Services Manager
    
    	Select [Control Panel] -> [Administrative Tools] -> [Services], and then stop WebOTX AS.
	- Method B: stopping the service using Command Prompt
	    
       		net stop "WebOTXAS10.xAgentService"

    (III)  Deleting domain
    
    Go to <INSTALL_ROOT> on  the command prompt and run the following command to remove the WebOTX AS domain. The environment variable JAVA_HOME must be set for the installed JDK.
        
        .¥bin¥asant -f setup.xml uninstall
    If the WebOTX AS domain has been deleted, "BUILD SUCCESSFUL" will be displayed on the command prompt.

2. Recreating the WebOTX AS domain1 [N1]

    Create a WebOTX AS domain on the switching partition of disk resource.
    In N1, move to <INSTALL_ROOT> on the command prompt and execute the following command.

        .¥lib¥ant¥bin¥ant -f setup.xml -Ddomains.root=Z:¥¥domains setup
    
3. Starting domain1

    In N1, move to <INSTALL_ROOT> on the command prompt and execute the following command to start domain1.

    	.¥bin¥otxadmin start-domain --domaindir Z:¥domains domain1

### Setting WebOTX AS environment  
1. Setting the floating IP address for ObjectBroker [N1]

    Select [Application Server]-[ORB Config] from the tree on the left side of WebOTX Administration Tool. In the [Common] tab, change [NameServiceHostName] and [ExternalHostName] to the floating IP address (192.168.1.111).

2. Setting a floating IP address for JMS [N1]

    Select [Application Server]-[JMS service]-[JMS host]-[default_JMS_host] from the tree on the left side of WebOTX Administration Tool, and change the host name in the [General] tab to the floating  IP address (192.168.1.111).

3. Setting a floating IP address setting for TP system [N1]

    Select [TP System] from the tree on the left side of WebOTX Administration Tool, select [System information] tab and change [Connection server name] and [Host name of name server] to the floating IP address (192.168.1.111).

4. Setting JNDI service [N1]

    Select [System] -> [System Settings] and change [Attribute Display Level] to "Display Detail Level Information".
    After that, set [JNDI server identification name] in [Application Server]-> [JNDI Service]-> [General] in the WebOTX Administration Tool to "aps1jndi".

5. Stopping domain [N1]
    
    In N1, move to <INSTALL_ROOT> on the command prompt and execute the following command to stop domain1 and domain2.
        
        .¥bin¥otxadmin stop-domain --domaindir Y:¥domains domain1

6. Deleting ObjectBroker name server persistent information [N1]
        
    Delete the following file.

    	Z:¥domains¥domain1¥config¥ObjectBroker¥namesv.ndf

7. Setting a floating IP address for the transaction service [N1]

    Open Z:\domains\domain1\config\TS\jta.conf in an editor and add the following definition under the JTA session.

    	LogicalHostname = "floating IP or virtual hostname"

8. Registering the web server service [N2]

    If you are using a WebOTX web server, register the WebOTX web server service on N2. Move the failover group to N2 so that you can see the switching partition Z on N2, and then run the following command.

    	Z:¥domains¥domain1¥bin¥apachectl INSTALL

9. Creating a WebOTX configuration file [N2]

    Create a configuration file to operate the WebOTX AS domain on the switching partition from N2. Execute the following command on N2.

    	<INSTALL_ROOT>¥lib¥ant¥bin¥ant -f setup.xml -Ddomains.root=Z:¥¥domains setup.env.client setup.env.server

10. Registering domain information in the TP system [N2]

    Register the domain information on the switching partition to the TP system with N2. On the command prompt, go to <INSTALL_ROOT>\Trnsv\bin and execute the following command. Set the value of the TPM option argument to the domain name.

    	contps -i AD TPM=domain1 CAT=Z:¥¥domains¥¥domain1¥¥config¥¥tpsystem WAIT=30

11. Changing the method of starting the WebOTX service [N1, N2]

    Change the startup method of WebOTX AS 10.x Agent Service to manual on N1 and N2.
On the [Control Panel]-[Administrative Tools]-[Services] screen, right-click WebOTX AS 10.x Agent Service, select Properties, and then change the startup type to "Manual".

### Editting EXPRESSCLUSTER start/stop script
Edit the EXPRESSCLUSTER start / stop script and set the monitoring settings.
1. Edit start / stop script
	- Edit the start / stop script by referring to the script resource item described in the EXPRESSCLUSTER X manual.
- Sample script
	- The following is a sample script resource to be registered in EXPRESSCLUSTER. Add the part in bold.
- Start script
<pre>
rem *************
rem "business normal processing"
rem *************
<b>
rem "Start WebOTX AS domain"
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin start-domain admin
</b>
rem priority check
IF "%CLP_SERVER%" == "OTHER" GOTO ON_OTHER1

rem *************
rem "Startup and recovery process after failover"
rem *************

<b>
rem "Start domain" 
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin start-domain admin
</b></pre>

- Stop script
<pre>
rem *************
rem "Business as usual"
rem *************
<b>
rem "Stop WebOTX AS domain"
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin stop-domain --force --wait_timeout 300 domain1
call otxadmin stop-domain --force --wait_timeout 300 admin
</b>

rem priority check
IF "%CLP_SERVER%" == "OTHER" GOTO ON_OTHER1

rem *************
rem "Startup and recovery process after failover"
rem *************
<b>
rem "Shut down WebOTX AS domain"
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin stop-domain --force --wait_timeout 300 domain1
call otxadmin stop-domain --force --wait_timeout 300 admin
</b>
</pre>

### Setting monitor resource for WebOTX monitoring
1. The following procedure configures monitoring settings using the WebOTX monitoring resource. If you have not registered the license for the WebOTX monitoring resource of EXPRESSCLUSTER X, register the license from the license manager of EXPRESSCLUSTER X. [N1, N2]
#### [Reference product manual]
 - EXPRESSCLUSTER X Installation & Configuration Guide
   - Chapter 5 Registering a License
2. Refer to the EXPRESSCLUSTER X manual and register the WebOTX monitoring resource.
#### [Reference product manual]
 - EXPRESSCLUSTER X Reference Guide
   - Chapter 4 Monitor Resource Details
     - Understanding WebOTX monitoring resources

The user name and password of the WebOTX admin user are set as follows by default.  
 - Username: admin  
 - Password: adminadmin  

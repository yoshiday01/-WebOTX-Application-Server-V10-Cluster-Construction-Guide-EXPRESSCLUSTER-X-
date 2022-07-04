# WebOTX Application Server V10 Cluster Construction Guide  ( EXPRESSCLUSTER X )

## Preface
This document describes the procedure in order to create a 2 nodes Active-standby cluster of WebOTX Application Server 10.x (WebOTX AS) by EXPRESSCLUSTER X 4 on Windows.

## Determing a system configuration
### Applicable software
OS 
- Windows Server 2012
- Windows Server 2012 R2
- Windows Server 2016
### Other software
- WebOTX Application Server V10.x
- EXPRESSCLUSTER X 4.x for Windows
- Java SE 8/11

## Configuring a cluster system
Install EXPRESSCLUSTER X and WebOTX AS on each node by following the procedures in the product manuals.
### [Reference product manual]
- EXPRESSCLUSTER X Installation & Configuration Guide
    - Chapter 4 Installing EXPRESSCLUSTER
- WebOTX Application Server Setup Guide

After installing EXPRESSCLUSTER X 4.x and WebOTX AS, please create a cluster by following the procedures.

## Properties file for domain creation
This section describes a sample of the properties file for domain creation (domain name.properties). When creating a domain, note the following:

1.	Set the different port number for each domain to avoid port confliction.
2.	Make sure that the port numbers used are not the same, including the domain you create separately.
3.	Change the ID and password for managing the domain as necessary.

The domain-name.properties (domain1.properties by default) is created in <INSTALL_ROOT> during installation. If you create multiple domains, copy the file for domain creation. Please note the above three points, create the domain creation file, and then create the domains.

- domain1.properties
```
domain.hostname = localhost
domain.name = domain1						        <- Set the name of the domain
domain.admin.user = admin 				                <- Set the domain admin user name
domain.admin.password = adminadmin 			         	<- Set the password for the domain admin user
domain.admin.port = 6212
domain.admin.jmxmp.port = 6712
domain.http.port = 80
domain.https.port = 443
domain.http.admin.port = 5858
domain.http.adminrest.port = 20101
domain.http.ajp.port = 8099
domain.jms.port = 9700
domain.jms.user.port = 9701
domain.jms.admin.port = 9702
domain.java.debugger.port = 9010
domain.ipv6-enable = false
domain.embedded-iiop-service.port = 7780

#ObjectBroker Service Configs
server.corba-service.oadj.Port = 9826
server.corba-service.namesv.NameServicePort = 2809
server.corba-service.namesv.NameServiceRoundRobin = true
server.corba-service.oad.OadPort = 9825
 
### TPMonitorManagerService Setup Properties (Standard / Enterprise Edition only) ###
tpsystem.IIOPListener.listenerPortNumber = 5151
tpsystem.AJPListener.listenerPortNumber = 20102
```

- domain2.properties
```
domain.hostname = localhost
domain.name = domain2 					        	<- Set the name of the domain
domain.admin.user = admin 					        <- Set the domain admin user name
domain.admin.password = adminadmin					<- Set the password for the domain admin user
domain.admin.port = 16212
domain.admin.jmxmp.port = 16712
domain.http.port = 8081
domain.https.port = 8443
domain.http.admin.port = 15858
domain.http.adminrest.port = 30101
domain.http.ajp.port = 18099
domain.jms.port = 19700
domain.jms.user.port = 19701
domain.jms.admin.port = 19702
domain.java.debugger.port = 19010
domain.ipv6-enable = false
domain.embedded-iiop-service.port = 17780
 …
#ObjectBroker Service Configs
server.corba-service.oadj.Port = 19826
server.corba-service.namesv.NameServicePort = 12809
server.corba-service.namesv.NameServiceRoundRobin = true
server.corba-service.oad.OadPort = 19825
 …
### TPMonitorManagerService Setup Properties (Standard / Enterprise Edition only) ###
tpsystem.IIOPListener.listenerPortNumber = 15151
tpsystem.AJPListener.listenerPortNumber = 30102
```

## Cluster environment construction on Windows (A 2node Active-standby cluster environment)
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

## EXPRESSCLUSTER initial settings
Refer to the EXPRESSCLUSTER manual to set up a 2 node active-standby cluster environment. 
### [Reference product manual]
- EXPRESSCLUSTER X Installation & Configuration Guide       
    - Chapter 6　Creating the cluster configuration data

Configure a failover group with the following resources cluster, upload the configuration to the cluster and start the cluster with Cluster WebUI.   
This guide is a shared disk, but you can constructing a mirror disk as well.
```
+-----------------------------------------------------------------------+
|Failover group                                                         |
+-------------------------------+-------------------+-------------------+
|Floating IP resources          |  Resources name   | fip1              |
|				+-------------------+-------------------+
|                               |  IP address 	    | 192.168.1.111     |
+-------------------------------+-------------------+-------------------+
|Virtual computer name resource | Resources name    | vcom1             |
|				+-------------------+-------------------+
|	        		| Virtual host name | webotx1           |
+-------------------------------+-------------------+-------------------+
|Disk resource	                | Resources name    | sd1               |
|				+-------------------+-------------------+
|	                        | drive             | Z:                |
+-------------------------------+-------------------+-------------------+
|Script resource                | Resources name    | script1           |
|				+-------------------+-------------------+
|                               | script	    | start.bat,stop.bat|
+-----------------------------------------------------------------------+
                            Table2. sample values
```


## WebOTX AS domain creation
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

    (III)  Deletioning domain
    
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

## WebOTX AS environment settings
1. Setting the floating IP address for ObjectBroker [N1]

    Select [Application Server]-[ORB Config] from the tree on the left side of the operation management tool. In the [Common] tab, change [NameServiceHostName] and [ExternalHostName] to the floating IP address (192.168.1.111).

2. Setting a floating IP address for JMS [N1]

    Select [Application Server]-[JMS service]-[JMS host]-[default_JMS_host] from the tree on the left side of the operation management tool, and change the host name in the [General] tab to the floating  IP address (192.168.1.111).

3. Setting a floating IP address setting for TP system [N1]

    Select [TP System] from the tree on the left side of the WebOTX Administration Tool, select [System information] tab and change [Connection server name] and [Host name of name server] to the floating IP address (192.168.1.111).

4. Setting JNDI service [N1]

    Select [System] -> [System Settings] and change [Attribute Display Level] to "Display Detail Level Information".
    After that, set [JNDI server identification name] in [Application Server]-> [JNDI Service]-> [General] in the WebOTX Administration Tool to "aps1jndi".

5. Stopping domain [N1]
    
    In N1, move to <INSTALL_ROOT> on the command prompt and execute the following command to stop domain1 and domain2.
        
        .¥bin¥otxadmin stop-domain --domaindir Y:¥domains domain1

6. Deleting ObjectBroker name server persistent information [N1]
        
    Delete the following files.

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

10. Registring domain information in the TP system [N2]

    Register the domain information on the switching partition to the TP system with N2. On the command prompt, go to <INSTALL_ROOT>\Trnsv\bin and execute the following command. Set the value of the TPM option argument to the domain name.

    	contps -i AD TPM=domain1 CAT=Z:¥¥domains¥¥domain1¥¥config¥¥tpsystem WAIT=30

11. Changing the method of starting the WebOTX service [N1, N2]

    Please change the startup method of WebOTX AS 10.x Agent Service to manual on N1 and N2.
On the [Control Panel]-[Administrative Tools]-[Services] screen, right-click WebOTX AS 10.x Agent Service, select Properties, and then change the startup type to "Manual".

## About EXPRESSCLUSTER start / stop script
Edit the EXPRESSCLUSTER start / stop script and set the monitoring settings.
1. Edit start / stop script
	- Edit the start / stop script by referring to the script resource item described in the EXPRESSCLUSTER X manual.
- Sample script
	- The following is a sample script resource to be registered in EXPRESSCLUSTER. Please add the part in bold
- Start script
```
rem *************
rem business normal processing
rem *************
rem Start WebOTX AS domain
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin start-domain admin
rem priority check
IF "%CLP_SERVER%" == "OTHER" GOTO ON_OTHER1
rem *************
rem Startup and recovery process after failover
rem *************
rem Start domain
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin start-domain admin
```

- Stop script
```
rem *************
rem Business as usual
rem *************
rem Stop WebOTX AS domain
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin stop-domain --force --wait_timeout 300 domain1
call otxadmin stop-domain --force --wait_timeout 300 admin
rem priority check
IF "%CLP_SERVER%" == "OTHER" GOTO ON_OTHER1
rem *************
rem Startup and recovery process after failover
rem *************
rem Shut down WebOTX AS domain
set PATH=%PATH%;C:¥WebOTX¥bin
call otxadmin stop-domain --force --wait_timeout 300 domain1
call otxadmin stop-domain --force --wait_timeout 300 admin
```

## Definition of monitor resource for WebOTX monitoring
1. The following procedure configures monitoring settings using the WebOTX monitoring resource. If you have not registered the license for the WebOTX monitoring resource of EXPRESSCLUSTER X, register the license from the license manager of EXPRESSCLUSTER X. [N1, N2]
### [Reference product manual]
 - EXPRESSCLUSTER X Installation & Configuration Guide
   - Chapter 5 Registering a License
2. Refer to the EXPRESSCLUSTER X manual and register the WebOTX monitoring resource.
### [Reference product manual]
 - EXPRESSCLUSTER X Reference Guide
   - Chapter 4 Monitor Resource Details
     - Understanding WebOTX monitoring resources

The user name and password of the WebOTX admin user are set as follows by default.

    Username: admin
    Password: adminadmin

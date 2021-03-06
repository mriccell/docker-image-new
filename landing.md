Providing this WebLogic Server image facilitates the configuration, and environment setup. This project includes the installation and the creation of an empty WebLogic Server domain (only an Admin Server). These WebLogic Server 12.2.1.3 images are based on Oracle Linux and Oracle JDK 8 (Server JRE).
For more information on WebLogic Server certification please check the [Oracle WebLogic Server on Docker Certification Whitepaper](http://www-content.oracle.com/content/groups/public/@otn/documents/webcontent/3090570~1.docx) and [WebLogic Server Blog](https://blogs.oracle.com/weblogicserver/) for updates.
## Get Started
### Providing Admin server Usernasme and Password
The username and password must be supplied in a domain.properties file located in a HOST directory that you will map at Docker run time with a -v option. The properties file enables the scripts to configure the correct authentication for the WebLogic Administration Server.
The format of the domain.properties file is key value pair:
	
	username=myadminusername
	password=myadminpassword

**Note**: Oracle recommends that the domain.properties file be deleted or secured after the container and the WebLogic server are started so the username and password are not inadvertently exposed.
To create an empty domain with an Admin Server running, you simply call 

	$ docker run -d -p 7001:7001 -p 9002:9002  -v `HOST PATH where the domain.properties file is`:/u01/oracle/properties  container-registry.oracle.com/middleware/weblogic:12.2.1.3

The WebLogic Server image will invoke createAndStartEmptyDomain.sh as the default CMD, and the Admin Server will be running on port 9002. When running multiple containers map port 7001 and 9002 different ports on the host: 

To run a second container on port 7002: 

	$ docker run -d -p 7002:7001 -p 9004:9002 container-registry.oracle.com/middleware/weblogic:12.2.1.3

Now you can access the AdminServer Web Console at https://localhost:9002/console.  Your browser will request for you to accept Security Exception. To avoid the Security Exception you must update the WebLogic server SSL configuration with a custom identity certificate.
## Customize your own WebLogic Server Domain
You might want to customize your own WebLogic Server domain by extending this image. The best way to create your own domain is by writing your own Dockerfiles,and using [WebLogic Scripting Tool (WLST)](https://docs.oracle.com/middleware/1221/cross/wlsttasks.htm) to create clusters, Data Sources, JMS Servers, Security Realms, and deploy applications. 
In your Dockerfile you will extend the WebLogic Server image with the FROM container-registry.oracle.com/middleware/weblogic:12.2.1.3 directive.
We provide a variety of examples (Dockerfiles, shell scripts, and WLST scripts) to create domains, configure resources, deploy applications, and use load balancer in [GitHub](https://github.com/oracle/docker-images/tree/master/OracleWebLogic/samples).
## Customer Support
We support WebLogic Server in certified Docker containers, please read our [Support Statement](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=230541239953849&id=2017945.1&_afrWindowMode=0&_adf.ctrl-state=dbwgmftrb_4). For additional details on the most current WebLogic Server supported configurations please refer to [Oracle Fusion Middleware Certification Pages](http://www.oracle.com/technetwork/middleware/ias/oracleas-supported-virtualization-089265.html)
## Copyright
Copyright (c) 2014-2018 Oracle and/or its affiliates. All rights reserved. 

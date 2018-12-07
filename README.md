# Docker Images from Oracle

This repository stores Dockerfiles and samples to build Docker images for Oracle products and Open Source projects.

 - [Oracle Coherence](./OracleCoherence)
 - [Oracle Database](./OracleDatabase)
 - [Oracle Java JDK](./OracleJDK)
 - [Oracle HTTP Server](./OracleHTTPServer)
 - [Oracle Tuxedo](./OracleTuxedo)
 - [Oracle WebLogic](./OracleWebLogic)

And Open Source projects:

 - [GlassFish](./GlassFish)
 - [MySQL](https://github.com/mysql/mysql-docker/)
 - [NoSQL](./NoSQL)
 - [OpenJDK](./OpenJDK)

Oracle Linux images can be found in the [`OracleLinux-images`](https://github.com/oracle/docker/tree/OracleLinux-images) branch of this repository.

For support and certification information, please consult the documentation for each product.

        ADMIN_NAME=admin-server
        ADMIN_HOST=wlsadmin
        MANAGED_SERVER_NAME_BASE=managed-server
        CONFIGURED_MANAGED_SERVER_COUNT=2
        CLUSTER_NAME=cluster-1
        DEBUG_FLAG=true
        PRODUCTION_MODE_ENABLED=true
        CLUSTER_TYPE=DYNAMIC

**NOTE:** Before invoking the build make sure you have built `oracle/weblogic:12.2.1.3-developer`.

Under the directory `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/container_scripts` find the script setEnv.sh. This script extracts the following Docker arguments and passes them as a --build-arg to the Dockerfile.

* Domain Name:           `DOMAIN_NAME`         (default: `base_domain`)
* Admin Port:            `ADMIN_PORT`          (default: `7001`)
* Managed Server Port:   `MANAGED_SERVER_PORT` (default: `8001`)
* Debug Port:            `DEBUG_PORT`          (default: `8453`)
* Database Port:         `DB_PORT`             (default: `1527`)
* Admin Server Name:     `ADMIN_NAME`          (default: `admin-server`)
* Admin Server Host:     `ADMIN_HOST`          (default: `wlsadmin`)

**NOTE:** The DOMAIN_HOME will be persisted in the image directory `/u01/oracle/user-projects/domains/$DOMAIN_NAME`.

To build this sample, run:

        $ . container-scripts/setEnv.sh ./properties/docker_build/domain.properties
        $ docker build $BUILD_ARG  --force-rm=true -t 12213-domain-home-in-image .


**During Docker Run:** of the admin and managed servers, the user name and password need to be passed in as well as some optional parameters. The property file is located in a `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/properties/docker_run` in the HOST. In the Docker run command line add the -v option which maps the property file into the image directory /u01/oracle/properties.


To start the containerized Administration Server, run:

        $ docker run -d --name wlsadmin --hostname wlsadmin -p 7001:7001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker_run:/u01/oracle/properties \
          12213-domain-home-in-image

To start a containerized Managed Server (MS1) to self-register with the Administration Server above, run:

        $ docker run -d --name MS1 --link wlsadmin:wlsadmin -p 8001:8001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker_run:/u01/oracle/properties \
          -e MANAGED_SERV_NAME=managed-server1 12213-domain-home-in-image startManagedServer.sh


To start a second Managed Server (MS2), run:

        $ docker run -d --name MS2 --link wlsadmin:wlsadmin -p 8002:8001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker_run:/u01/oracle/properties \
          -e MANAGED_SERV_NAME=managed-server2 12213-domain-home-in-image startManagedServer.sh

The above scenario from this sample will give you a WebLogic domain with a cluster set up on a single host environment.

You may create more containerized Managed Servers by calling the `docker` command above

# Copyright
Copyright (c) 2014-2018 Oracle and/or its affiliates. All rights reserved.

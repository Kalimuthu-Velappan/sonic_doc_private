

# Third Party Container Management

## High Level Design Document
**Rev 0.1**

## Table of Contents

* [List of Tables](#list-of-tables)
* [Revision](#revision)
* [About This Manual](#about-this-manual)
* [Scope](#scope)
* [Definition/Abbreviation](#definitionabbreviation)
* [Requirements Overview](#requirements-overview)
  * [Functional Requirements](#functional-requirements)
  * [Configuration and Management Requirements](#configuration-and-management-requirements)
  * [Scalability Requirements](#scalability-requirements)
  * [Coldboot/Warmboot Requirements](#coldbootwarmboot-requirements)
* [Functional Description](#functional-description)
  * [TPCM and TPC Storage](#tpcm-and-tpc-storage)
  * [TPC Image Generation](#tpc-image-generation)
  * [TPCM Service](#tpcm-service)
  * [Resource Limitation and Configuration](#resource-limitation-and-configuration)
  * [TPCM CLIs](#tpcm-clis)
  * [TPC Install](#tpc-install)
  * [TPC Uninstall](#tpc-uninstall)
  * [TPC List](#tpc-list)
  * [TPC Docker Upgrade](#tpc-docker-upgrade)
  * [System Reboot](#system-reboot)
* [CLI](#cli)
* [KLISH CLI](#klish-cli)
* [Unit Test](#unit-test)

# List of Tables

[Table 1: Abbreviations](#table-1-abbreviations)

# Revision

Rev   |   Date   |  Author   | Change Description
:---: | :-----:  | :------:  | :---------
1.0   | 15/05/20 | Kalimuthu | Initial version
2.0   | 22/07/20 | Precy Lee | KLISH CLI
2.1   | 23/07/20 | Precy Lee | update tpcm uninstall and upgrade CLIs 
2.2   | 06/08/20 | Precy Lee | TPCM OC yang compliance
3.0   | 01/04/22 | Kalimuthu | TPCM over MGMT VRF 



# About this Manual

This document describes the Third Party Container Management(TPCM) framework in the SONiC Network Operating System(NOS). This feature helps the user to install and manages the Third Party Containers(TPC) in SONiC.

# Scope

This document describes the high level design details of Third Party Container Management framework that supports of  installation/uninstallation, configuration and upgrade of the TPC Dockers.  It also describes the tools and commands used for the TPC image management. The document covers that TPC's are from Docker containers and not from other sources like LXC, podman etc. 

# Definition/Abbreviation

### Table 1: Abbreviations

| **Term**     |  **Meaning**                  |
|:-------------|:------------------------------|
| TPC  | Third party container                 |
| TPCM | Third party container management      |
| ONIE | Open network installation environment  |
|  | |


# Requirements Overview

This document describes mechanisms for Third Party Container Management(TPCM) in  the SONiC environment. The TPCM allows the user to Install and Manage the TPC docker into the SONiC.  

SONiC is a network operating system based on the Debian distribution. The  SONiC components are tightly integrated with the installation image that are created during the build time itself. It allows the user to install the additional tools from the standard Debian repo, but any addition of new components to the SONiC system requires the rebuilding of the SONiC binary image. This feature allows user to install the third party components in the form of containers and that can be loaded and integrated as part of the SONiC system independently without rebuilding of SONiC image. 

SONiC provides the ability to pull and install Docker containers from any Docker Hub Registry using standard Docker pull/load commands, but it requires a utility in the SONiC to manage the external Docker containers. Docker consists of two parts 
- 1. Docker IMAGE - It is a read-only Docker image layer.
- 2. Docker CONTAINER - It is read-write Docker image layer on top of Docker IMAGE. 

Docker image can be tagged as multiple Docker images and for each image, one or more Docker containers can be created. Docker supports standard commands (add/update/remove) to manipulate the containers and image layers in the system. But it doesn't support the SONiC Docker management functions like Container restart upon system reboot, Container persistence and migration across SONiC image upgrades, Seamless container upgrade, service association for each container, Container System resource configuration, Monitoring, etc. The TPCM provides a framework to manage these functions in SONiC system.   

TPCM should support the following Docker life cycle:
	1. Docker image can be downloaded and installed/ docker can be pulled from docker registry.
	2. This will be installed in current SONIC FS.
	3. Docker container creation and running container.
	4. When the docker runs, can store data into its file system or mounted FS.
	5. Docker snapshot can be saved as image to be exported elsewhere.
	6. Docker stop/ remove/ of container.
	7. Removal of docker image.
	8. Upgrade of an existing docker container.

## Functional Requirements

### TPCM Framework Requirements

- Provide a mechanism for customers to bring their custom Docker container(TPC) into the SONiC OS environment.
- TPCM framework should provide an interface to install, uninstall, upgrade and list the TPC in the SONiC system.
- It should support that zero or more TPC can be installed on the SONiC system.
- TPC image upgrade shall be independent of SONiC image upgrade.
- SONiC image upgrade shall migrate all the currently installed TPCs into the newly installed image seamlessly. 
- TPCM should provide systemd service support for each of the TPC installed on the system.
- All the TPCs should get started automatically upon fast/warm/cold reboot
- TPCM should provide support for installation and upgrade of TPC from multiple sources - http server, scp/sftp server, local media and external Docker registry.
- TPCM should provide support for seamless TPC upgrade with the provision of container data backup and restore. 
- TPCM should provide an interface to enforce the TPC resource[ DISK, CPU and MEMORY ] usage limitation, but the current release covers only the CPU limitation. 


## SONIC 3.7 - TPC over Static VRF Requirements(4/22 - Code Drop)
- Provision the TPCM to run on the statically configured Mgmt-VRF.
- Resource management for CPU and Memory.
- Provision TPCM to pass the container startup arguments.
- Dedicated TPC storage management for VRF based dockerd and containerd instances.
- Image migration support for downgrading TPC from 3.7 to its prior release.
- KLISH/CLICK/REST API changes for VRF integration
- TPC docker migration between Mgmt-VRF and default VRF.
- TPCs are allowed to run only on either default VRF or mgmt-VRF and any other VRF name should be rejected.

## SONIC 3.7 - TPC over Dynamic VRF Requirements(4/29 - Code Drop)
- Provision the TPCM to run on the dynamically configured Mgmt-VRF.
- TPC docker migration between Mgmt-VRF and default VRF when Mgmt VRF is configured/unconfigured. 
	- If the mgmt-VRF is deleted with running TPC on it, it should automatically migrate all the VRF TPCs into default VRF.
	- When mgmt-VRF is configured again, it should automatically migrate all VRF TPSs from default VRF to mgmt-VRF.
	- If TPCs are not configured with VRF, it should always run in the default VRF.


## Configuration and Management Requirements

- In order to integrate the TPC with SONiC, a set of config and show commands shall be supported.

### Config commands
- Clik and Klish Config commands shall be supported for TPCM to add, remove and upgrade of TPC container into SONiC.

### Show commands
- Show commands to display the list of installed TPC containers 
 
## Scalability Requirements
- Since the TPC folder is shared with SONiC partition, it adds the limitation to the amount of data stored on the SONiC partition. 
- Hardware resources(such as memory, CPU and data Volume mount) usage limit is enforced through predefined configuration for all the TPC Dockers.

## Coldboot/Warmboot Requirements
- The system should auto start all the installed TPC Dockers.

# Functional Description


## TPCM and TPC Storage

- All the TPCs and it's Image layers are stored as part of SONiC Docker image file system. The Docker image FS storage mostly located on the /var/lib/docker/overlay2 folder which is specific to each SONiC image in the system. A new TPCM folder is created in the SONiC partition for storing all the TPC container management files. The TPCM storage is shared across all the SONiC image in the system. The TPCM storage mainly contains the following contents.

1. TPC systemd service files.
2. Private data volume for each TPC.
3. One Shared data volume for all the TPCs.
4. Resource configuration file for each TPC.
5. TPCM service files and utility scripts that gets installed as part of SONiC installation.
6. Used as temporary storage for storing the TPC container image during SONiC image upgrade.

## TPCM and TPC Storage - Management VRF
- All the TPCs and it's Image layers are stored as a separate Docker image in the host file system. 
- The Docker image FS storage mostly located on the /host/tpcm/mgmt/docker/overlay2 folder which is specific to common all the SONiC image in the system. 
- A new TPCM folder is created in the /host/tpcm/ directory for storing all the TPC container management files.
- The/host/tpcm/ storage is shared across all the SONiC image in the system. 

#### The TPCM storage mainly contains the following contents.

1. TPC docker and container images. 
2. TPC systemd service files.
3. Private data volume for each TPC.
4. One Shared data volume for all the TPCs.
5. Resource configuration file for each TPC.
6. TPCM service files and utility scripts that gets installed as part of SONiC installation.
7. Used as temporary storage for storing the TPC container image during SONiC image upgrade.

## TPC Image Generation

- Docker image can be created using one of the following methods:
	1. Pulled directly from an external Docker Hub Registry.
	2. Imported/Loaded from a Docker image file which got created using docker export/save command from an existing docker image and it will in the form of compressed tar format.
	3. Created from a custom Docker Build file using Docker build utilities.
	4. Created from an existing Docker container by saving the container image as a new Docker image.

## TPCM Service

- The TPCM service is a SONiC systemd service and it gets added as part of SONiC during build itself. 
- This service takes care of migration of running containers into the newly installed image. 
- During the SONiC image upgrade, it saves all the running containers into TPC storage folder. After reboot, it restores all the TPCs into the newly loaded image.

## tpcm@mgmt Service
The TPCM@mgmt service is a SONiC systemd service and it gets added dynamically when MGMT VRF is configured.
- This service takes care of starting the docker@mgmt.service and container@mgmt.service instances. 
- The @mgmt.services launch the seconds instance of dockerd and containerd instance under mgmt VRF.  
- Since it uses shared storage for TPC, No change is required during the SONiC image upgrade and reload. 

## @mgmt.services
- When mgmt VRF is configured two @mgmt services gets created and lanuched automatically.
- It creates the dockerd@mgmt.servce and containerd@mgmt.service instances.
- The container@mgmt.server creates second instance of containerd over mgmt cgroup.
- The dockerd@mgmt.service creates the seconds of dockerd over cgroup and connects to second containerd instance through unix socket. 
- Communication between the between @mgmt services are through mgmt named unix socket.
- These servies are removed automatically when mgmt VRF is removed.


## Resource Limitation and Configuration 
### Disk Storage
- TPCM feature will be supported on systems with a minimum of 32G.
- TPC with the System Docker
    - To support auto migration of TPCM dockers, we need double the size and hence only half of it can be effectively used.

- TPC Resource Limitation with MGMT VRF.
    - TPC dockers are shared across SONiC images, so this doesnt require temporary storage.

### CPU Resource
- By default, for each TPC docker, the CPU resource limit is restricted to 20%.

### Memory Resource
- Memory is restricted to 30% of the system memory.


## TPCM CLIs

- TPCM related operations are implemented through a CLI called 'tpcm'. Set of options with optional parameters shall be provided to perform TPCM operations. 

- Supported operations:
		
		tpcm install   => Install the TPC image into SONiC system.
		tpcm uninstall => Uninstall the TPC image from SONiC system.
		tpcm upgrade   => Upgrade the existing TPC container.
		tpcm list      => list the existing TPC images.


## TPC Install

- The TPCM install option installs the TPC container into SONiC system. The container image can be created in two forms, either from an image file or pulled from an externel Docker registry. 
- The TPC image can be installed into SONiC system by using one of the following sources:
	- HTTP SERVER - Installed from http server.
	- SCP PATH    - Copied from a remote server through scp protocol.
	- SFTP SERVER - Copied from a remote server through sftp protcol.
	- MEDIA PATH  - Copied from a local media folder.
	- DOCKER HUB  - Downloaded from remote Docker registry.
	- LOCAL IMAGE - Use one of the existing docker image. 

- One or more TPC images can be installed on the SONiC system. Once the SONiC OS is up, the TPC instance can be brought up into SONiC by running tpcm install command. 

- Example:

		tpcm install tpc-name /path/docker-tpcimage.gz -y

- While installing the TPC image, it creates the sonic service for each of the TPC's to bring up the TPC container up.  So the loading of the TPC container image is needed only once, On the subsequent reboot, these TPC services will be started automatically by the systemd manager in the SONiC system.
- The TPC service file is generated for each TPC container. The generated service is stored under tpcm storage folder and also installed on the systemd service folder.

- During the installation process, the following steps get executed.
 1. Download the TPC image from one of the sources.
 2. Install the TPC image into SONiC system.
 3. Create a systemd service and resource configuration for the newly installed TPC image.
 4. Create a container for the newly installed TPC image. 
 5. Bring up container instance by starting the TPC service. 
 
## TPC Uninstall

- The uninstallation option will remove the TPC container from the SONiC system. 

	- Example:
		
			tpcm uninstall <tpc-name>  -y
		
- During the uninstallation process, the following steps get executed.

 1. Stops the running TPC container by stopping the corresponding TPC service.
 2. Removes the TPC service file from the SONiC systemd service manager.
 3. Removes the TPC from the SONiC system.
 4. Removes the TPC image from the SONiC system
 
-  Once the TPC and its associated service is removed, the subsequent SONiC reboot will not auto start the removed TPC services.

## TPC List

- The TPCs can be listed using the tpcm list command. 
		
		# tpcm list 

### Running TPC Dockers

- The loaded TPC can be viewed using the following regular Docker command to view the running Dockers. This command will list all the running containers including the SONiC as well as TPC Docker containers.

		# docker ps 
		
## TPC Docker Upgrade

- The TPCM framework allows the user to upgrade the running TPC Dockers.  The tcpm framework upgrade option allows the user to upgrade an existing TPC docker with a new one. During the upgrade process, TPCM provides the interface to run the pre/post script present in the container. These hook allow the user to take the backup of data from existing TPC container and restore it back into the upgraded TPC Docker. Each TPC is provided with private data volume for storing the container data which will presist across docker upgrade. The pre/post hook is an optional script that should be added by the user and it will be based on the user needs on what data to be backed up during the upgrade. If all the TPC's data is already stored on the private volume or user doesnt want to take data backup during upgrade then this script is optional. 

	- The pre hook is executed as part of running container and the post hook is executed as part of the upgraded container. 
    - These scripts are optional and it get executed only if it is present in the container. It skips the script execution if it is not present. 
    - Two level of hook script is supported.
        1. Internal hook scripts
            - These scripts are prepared by the user and it should come along with the container.
	        - The internal hook scripts should be placed in a fixed folder in the TPC container with the name as /tpc/scripts/pre.sh' and '/tpc/scripts/post.sh'. 
            - These scriptes get executed only if it is present inside the container.

        2. External hook scripts
            - These scripts are prepared by the user and stored in the host folder and volume mount into TPC. 
            - It will presist across TPC upgrade. 
	        - It should be placed in a fixed folder in the Host with the name as /tpcm/tpc/<tpc-name>/pdata/scripts/pre.sh' and '/tpcm/tpc/<tpc-name>/pdata/scripts/post.sh'. 
	        - As part of volume mount, it automatically get mapped into the TPC with the name as /tpc/pdata/scripts/pre.sh' and '/tpc/pdata/scripts/post.sh'. 
	- The pre script should create a backup data file and store them into '/tpc/pdata/' folder inside the container, and the post script should restore the contents from the data file. 
	- For each TPC, a private volume '/tpc/pdata' is mounted from host partition '/tpcm/tpc/\<tpc-name\>' for saving the data across TPC upgrade.

- Example:

		# tpcm upgrade  name <tpc-name> <tpc-image source> -y 
 
- It executes the following sequence to upgrade the running TPC Docker.
	1. Run the TPC internal pre hook script.
	2. Run the TPC external pre hook script.
	3. It stops the already running TPC container.
	4. Remove the existing TPC container.
	5. Loads the new TPC image into the Docker image list.
	6. Create the new TPC container from newly loaded TPC image.
	7. Run the newly created container.
	8. Execute the TPC external post script.
	9. Execute the TPC internal post script.
	10. Remove the old TPC image.

- During the TPC upgrade if any these operation fails, it will rollback to old container.


# CLI:

## 1. Installing the TPC image

TPC image can be loaded and installed in three forms:

#### TPC image from a file:
- It uses the download protocol to download the TPC image into a temporary path and then that gets loaded into Docker image filesystem. 
1. Load the TPC image file from a external http/https server

    - Syntax:
	    - #### tpcm install name \<tpc-container-name\> url \<URL\> [--vrf \<vrf-name\>] [--args \<docker args\>] [--cargs \<container args\>][-y]
	- where:
	    -  \<tpc-container-name\>  => Name of the TPC container to be loaded.
	    - \<URL\> => It is http/https web url from which the TPC image can be downloaded.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
        - [-y]  => User confirmation

	- Example:
    
	        tpcm install name mydocker url http://myserver/path/mydocker.tar.gz -y

2. Load the image file from a external server through scp/sftp

    - Syntax:
        - #### tpcm install name \<tpc-container-name> scp  \<server name>  --username \<username> [--password \<password>] --filename \<TPC image path> [--vrf \<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [-y] 
        - #### tpcm install name \<tpc-container-name> sftp \<server name>  --username \<username> [--password \<password>] --filename \<TPC image path> [--vrf \<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [-y] 
        
    - where:
        - \<tpc-container-name>  => Name of the TPC container to be loaded.
        - \<server name>    => Specifies the remote server name from which image is being download.
	    - --username  => Specifies the remote server user credentials.
	    - --password  => Specifies the remote server user's password. If passwd is not given, prompted passwd for during the image download.
	    - --filename  => Path on the remote server where the TPC image is located.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
	    - [-y] => User confirmation

    - Example:

            tpcm install name mydocker scp myserver  --username myuser --filename /path/mydocker.tar.gz -y
            Password:

3. Load the image file from a local media path

    - Syntax:
        - #### tpcm install name \<tpc-container-name> file  \<TPC image path> [--vrf \<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [-y]

    - where:
	     - \<TPC image path> => Local media path where the TPC image is located.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
        - [-y]   => User confirmation

	- Example:

			tpcm install name mydocker file /media/usb/path/mydocker.tar.gz -y

		
4. Load the TPC image from a external Docker repo

    - It loads the TPC image directly from external Docker registry into local Docker image filesystem.

	- Syntax:
	    - #### tpcm install name \<tpc-container-name> pull \<Image name>[:\<tagname>] [--vrf \<vrf-name>] [--args \<docker args>] [--cargs \<container args>] [-y]

	- where 
	    - <tpc-container-name>  => Name of the TPC container to be loaded.
	    - <Image name>  => Name of the image to be downloaded.
	    - [tagname]  => Optional, Tagname of the image, by default 'latest' would be taken. 
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
        - [-y] => User confirmation.

    - Example:

			tpcm install name mydocker pull ubuntu:latest -y

5. Use one of the existing docker image

	- It uses one of the exising docker images installed in the system. It simply creates the service file and takes care of TPC and migration feature across the image  upgrade. This allows the user to use the regular docker commands to create the TPC images. Once the image is ready, user can associate TPCM management feature to it. 

	- Syntax:
	    - #### tpcm install name \<tpc-container-name> image \<Image name>[:\<tagname>] [--vrf \<vrf-name>] [--args <docker args>] [--cargs <container args>] [-y]

	- where 
	    - \<tpc-container-name>  => Name of the TPC container to be created.
	    - \<Image name>  => Name of the docker image in the local system.
	    - tagname  => Optional, Tagname of the image, by default 'latest' would be taken. 
        - \<vrf-name> => VRF name
	    - --dargs => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
        - [-y] => User confirmation
        
    - Example:

			tpcm install name mydocker image ubuntu:latest -y
	
## 2. Uninstalling the TPC image

- Uninstalling the tpc image

    - Syntax
        - #### tpcm uninstall name \<container-name\> [--vrf \<vrf-name>] [-y] [--clean_data]

    - where 
	    - \<tpc-container-name\>  => Name of the TPC container to be removed.
        - \<vrf-name> => VRF name
	    - --clean_data  => It removes all the container specific data which includes config file, service file and container private data.
	    - [-y] => User confirmation

    - Example:

		    tpcm uninstall name mydocker -y

## 3. Upgrading the TPC image

- Upgrading the TPC image
    - Syntax
        - #### tpcm  upgrade  name \<container-name\> \<same options as install\> [--skip_data_migration]

    - where 
	    - --skip_data_migration => If yes, container data is not migrated or retained, else data is migrated and retained.  
	    All other options are same as install command.

	- Example:

			tpcm upgrade name mydocker image ubuntu:latest -y
	

## 4.  List the TPC images

- List the all TPC image that are installed on the system
    - Syntax
        - #### tpcm list [--vrf \<vrf-name>]
    - Where
        - \<vrf-name> => VRF name

- Example
            
		#  tpcm list
		CONTAINER NAME  IMAGE TAG         VRF             STATUS
		TEST            mydocker:latest   default         Up 8 seconds

## 5. TPC Service startup dependency
	
- TPC services are started in parallel with SONIC services.
- TPC should have capabilities to start after certain SONIC services.
- Custom config/service file to pass or modify the TPC service arguments


# KLISH CLI:

## 1. Installing the TPC image

#### TPC image from a file:

1. Load the TPC image file from a external http/https server

    - Syntax:
        #### tpcm install name \<tpc-container-name> url \<URL> [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]

        - Example:
            - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz
            - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz dargs " -e TESTENV=TESTVALUE"

2. Load the image file from a external server through scp/sftp

    - Syntax:
         - #### tpcm install name \<tpc-container-name> scp \<server name>  username \<username> password \<password> filename \<TPC image path> [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]
        - #### tpcm install name \<tpc-container-name> sftp <server name>  username \<username> password \<password> filename \<TPC image path> [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]

    - Example:

        - sonic# tpcm install name mydocker scp myserver  username myuser password paswd filename /path/mydocker.tar.gz

3. Load the image file from a local media path

- Load from the local file system
    - Syntax:
        - #### tpcm install name \<tpc-container-name> file  \<TPC image path> [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]

    - Example:
        - sonic# tpcm install name mydocker file /media/usb/path/mydocker.tar.gz
        - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz args " -e TESTENV=TESTVALUE"


4. Load the TPC image from a external Docker repo

- Load from the docker hub.
    - Syntax:
        - #### tpcm install name \<tpc-container-name> pull \<Image name>[:\<Tagname>] [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]

    - Example:

            sonic# tpcm install name mydocker pull ubuntu:latest

5. Use one of the existing docker image

    - Syntax:
        - #### tpcm install name \<tpc-container-name> image \<Image name>[:\<Tagname>] [vrf \<vrf-name>] [args \<docker args>] [cargs \<container args>]
        
    - Example:

            sonic# tpcm install name mydocker image ubuntu:latest

#### Data Model:
        module: openconfig-system-ext

            rpcs:
             +---x tpcm-install
             |  +---w input
             |  |  +---w docker-name?     string
             |  |  +---w image-source?    string
             |  |  +---w image-name?      string
             |  |  +---w remote-server?   string
             |  |  +---w username?        string
             |  |  +---w password?        string
             |  |  +---w args?            string
             |  +--ro output
             |     +--ro status?          int32
             |     +--ro status-detail*   string



#### Rest API Support:

      POST "<REST-SERVER:PORT>/restconf/operations/openconfig-system-ext:tpcm-install
      request body:
            {"openconfig-system-ext:input":{"docker-name":"string","image-source":"string","image-name":"string","remote-server":"string","username":"string","password":"string","args":"string"}}
      Examples:
         {"openconfig-system-ext:input":{"docker-name":"mydocker","image-source":"scp","image-name":"/path/mydocker.tar.gz","remote-server":"myserver","username":"myuser","password":"passwd","args":"-e TESTENV=TESTVALUE"}}

         {"openconfig-system-ext:input":{"docker-name":"mydocker","image-source":"file","image-name":"/path/mydocker.tar.gz","remote-server":"string","username":"string","password":"string","args":"-e TESTENV=TESTVALUE"}}


## 2. Uninstalling the TPC image

#### tpcm uninstall name \<container-name\> [vrf \<vrf-name>] [clean_data \<yes/no>]

- where 
	--clean_data => If yes, container data is removed, else data is not removed.  
	All other options are same as install command.

#### Data Model:

         module: openconfig-system-ext
          
         rpcs:
            +---x tpcm-uninstall
               +---w input
               |  |  +---w clean-data?    string
               |  |  +---w docker-name?   string
               |  +--ro output
               |     +--ro status?          int32
               |     +--ro status-detail*   string


#### Rest API Support:
      POST "<REST-SERVER:PORT>/restconf/operations/openconfig-system-ext:tpcm-uninstall
         request body:
           {"openconfig-system-ext:input":{"clean-data":"string","docker-name":"string"}}

         Examples:
            {"openconfig-system-ext:input":{"clean-data":"yes","docker-name":"mydocker"}}
            {"openconfig-system-ext:input":{"clean-data":"no","docker-name":"mydocker"}}



## 3. Upgrading the TPC image

#### tpcm  upgrade  name \<container-name\> <same options as install\> [skip_data_migration <yes/no>] [args \<docker args>] [cargs \<container args>]

        - Example:

           sonic#  tpcm upgrade name mydocker image ubuntu:latest
           sonic#  tpcm upgrade name mydocker sftp myserver username myuser password passwd filename mydocker.tar.gz skip_data_migration yes
 args " -e TESTENV=TESTVALUE"

#### Data Model: 

        module: openconfig-system-ext

            rpcs:
             +---x tpcm-install
             |  +---w input
             |  |  +---w docker-name?     string
             |  |  +---w image-source?    string
             |  |  +---w image-name?      string
             |  |  +---w remote-server?   string
             |  |  +---w username?        string
             |  |  +---w password?        string
             |  |  +---w skip-data-migration?   string
             |  |  +---w args?            string
             |  +--ro output
             |     +--ro status?          int32
             |     +--ro status-detail*   string



#### Rest API Support:

      POST "<REST-SERVER:PORT>/restconf/operations/openconfig-system-ext:tpcm-upgrade
         request body:
            {"openconfig-system-ext:input":{"docker-name":"string","image-source":"string","image-name":"string","remote-server":"string","username":"string","password":"string","skip-data-migration":"string","args":"string"}}

         Example:
             {"openconfig-system-ext:input":{"docker-name":"mydocker","image-source":"scp","image-name":"/path/mydocker.tar.gz","remote-server":"myserver","username":"myuser","password":"passwd","skip-data-migration"  : "yes", "args":"-e TESTENV=TESTVALUE"}}



## 4.  List the TPC images

#### show tpcm list [vrf \<vrf-name>] 
        - The TPCs can be listed using the tpcm list command.

        - Example:
           sonic#   show tpcm list
                    CONTAINER NAME  IMAGE TAG         VRF             STATUS
                    TEST            mydocker:latest   default         Up 8 seconds

#### Data Model:

        The openconfig-tpcm yang model is included as an extension to the openconfig-system yang model.

             +--rw oc-sys-ext:tpcm
               +--ro oc-sys-ext:state
                 +--ro oc-sys-ext:tpcm-image-list*   string

#### Rest API Support:

        GET "<REST-SERVER:PORT>/restconf/data/openconfig-system:system/openconfig-system-ext:tpcm/state/tpcm-image-list"

# TPCM with Mgmt VRF 
- Creating a management VRF will enslave eth0 under a layer 3 construct (VRF). 
- This will ensure traffic exchanges between the third-party docker to any internal server is not advertised on the data ports.
- Mgmt-VRF configuration:
	- All the SONiC dockers are untouched and it will remain run in the default VRF.
    - It restarts the tpcm@mgmt.service.
    - When the tpcm@mgmt.service is restarted, it checks the config DB for mgmt VRF existance.
    - If VRF is exists on the DB, it creates the @mgmt services and launch them.
    - The @mgmt services creates the second instance of dockerd and containerd over mgmt cgroup.
    - It migrates TPC into VRF storage and restarts all the TPC that were configured to run in mgmt VRF.
    - Any new TPC that are created with vrf option will go into VRF storage.
    - Any new TPC that are created without vrf option will go into default VRF.
- Mgmt-VRF unconfiguration:
    - TPCs that are already running on the default VRF are remain unchanged.
    - TPCs that running on the mgmt VRF are migrated to default VRF and gets restarted.
    
## TPC and DOCKER Interface.
	- Multiple instanace of dockerd and containerd will be running on the system.
    - In order to connect the TPC with right instance of dockerd and containerd, it uses the named unix socket.
    - It uses the VRF name as unix socket name for communication between TPC and dockerd/containerd.
    - If VRF is not configured, it uses the default unix socket which will connect to default VRF dockerd/containerd instances.
    - All the TPCM commands are automatically wrapped with corresponding VRF unix socket name.
    - If user needs to connect to the VRF docker, one of the following option should be used.

	    - Use DOCKER_HOST to access the TPC dockers.
		    - export DOCKER_HOST=unix:///mgmt.socket
            - docker ps
	    - Use -H option to pass the socket option.
            - docker -H unix:///mgmt.socket ps 


## SONiC Image ONIE install

- When SONiC is being net-installed through ONIE, it wipes out the entire SONiC partition. So the user should take the backup of existing TPC contents and restore them back manually if required.  


## SONiC image upgrade 

- When SONiC is being upgraded, the following sequence gets executed.

 1. Save all the TPC container as a new TPC image layers.
 2. Export all the TPC image in the form of compressed tar file and store them into the TPCM storage folder
 3. After the reboot, restore all the Docker container into the newly loaded SONiC image.
 
 ### Images on the VRF storage.
 1. TPC are stored on the VRF storage are shared across SONiC image partitions, so TPC migrations are not required.

## System Reboot
- During the system bootup, the following sequence gets executed.

 1. Start the SONiC TPCM service as part for systemd startup
 2. When SONiC TPCM service startup script is being executed, it checks for TPC container files presence in the TPC folder, if yes, load all the TPC Dockers along with its associated service file into SONiC system and start all the TPC services.
 3. Else, graceful exit from the startup script.
 
 ### TPCs on the VRF storage.
 1. During system boot, tpcm@mgmt.services gets started automatically through hostcfgd service when mgmt VRF config is replayed from config DB.
 2. The tpcm@mgmt.service creats the @mgmt service and starts all the TPCs that has vrf configuration.

# Unit Test


  | SNO |  Unit Testcase 
 :------| :----------------------------------------------------
    1 | Verify the TPCM service gets created as part of SONiC.
    2 | Verify the TPC install from http/https server.
    3 | Verify the TPC install from a remote server using SCP protocol.
    4 | Verify the TPC install from a remote server using SFTP protocol.
    5 | Verify the TPC install from a local media path
    6 | Verify the TPC install from an external Docker registry.
    7 | Verify the TPC install with different parameter combinations.
    8 | Verify the TPC uninstall of running containers.
    9 | Verify the TPC upgrade with an already installed image.
    10| Verify the TPC uninstall and its associated service file removal.
    11| Verify the TPC service auto startup after reboot.
    12| Verify the SONiC image upgrade with TPC container migration.
    13| Verify the SONiC ONIE uninstall with TPC container removal.
    14| Verify the Hardware resource Disk/CPU/Memory limitations.
    15| Verify the TPC list of all TPC dockers.
    16| Verify the TPC service start/stop/restart.
    17| Verify the VRF creation and deletion.
    18| Verify the TPC creation over mgmt VRF.
    19| Verify the TPC deletion over mgmt VRF.
    20| Verify the CPU resource usage of TPC.
    21| Verify the Memory resource usage of TPC.
    22| Verify the Disk resource usgae of TPC.
    23| Verify the VRF TPCs start during bootup.
    24| Verify the Container argument option of TPC.
    25| Verify the VRF option in the KLISH/CLICK/REST interface.
    26| Verify the TPC upgrade on VRF TPCs.
    27| Verify the TPC migration between default VRF and mgmt VRF.


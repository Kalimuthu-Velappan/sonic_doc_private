

# Third Party Container Management

## High Level Design Document
**Rev 3.5**

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
  * [TPC Update](#tpc-update)
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
3.1   | 25/04/22 | Senthil G | TPCM Resource(Memory) Limit
3.2   | 27/04/22 | Senthil G | TPCM update CLI
3.3   | 13/05/22 | Senthil G | TPCM service dependency
3.4   | 15/06/22 | Senthil G | 4.1 release TPCM Disk limit and porting enhancements from 3.7
3.5   | 15/07/22 | Senthil G | 4.1 disk limit update provision and design details

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

## SONIC 4.1 - Porting of TPC Enhancements from release 3.7
- Porting of TPC mgmt vrf support
- Porting of TPC memory limit support
- Porting of TPC service startup dependency( start after system ready)
- Porting of TPC update CLI support

## SONIC 4.1 - TPC Disk space limit and Restriction of TPC on low end platforms
- Resource management for Disk
- TPCs should not be allowed to consume more than 20% of the overall disk space.
- Individual TPC disk limits are not supported.
- Prevent TPC support on low end platforms with disk space less than 32G.
- Automatic imposing of disk limits for TPCs on 4.1 image when upgraded from earlier releases. 
  However, if the disk utilization of the TPCs in earlier releases is greater than default overall TPC disk limit, 
  adjusted disk limit will be automatically set in order to support the TPCs running in older releases.
  This is to make sure of a non-disruptive upgrade.
- Removal of TPC disk limits while migrating from current 4.1 release to older releases
- Updating the TPC disk limit(users to set limit) should be supported for the below reason
  Allowing the users to set the disk limit means providing them the control based on their deployment scenarios. 


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
- This service takes care of starting the dockerd@mgmt.service and containerd@mgmt.service instances. 
- The @mgmt.services launch the seconds instance of dockerd and containerd instance under mgmt VRF.  
- Since it uses shared storage for TPC, No change is required during the SONiC image upgrade and reload. 

## @mgmt.services
- When mgmt VRF is configured two @mgmt services gets created and lanuched automatically.
- It creates the dockerd@mgmt.servce and containerd@mgmt.service instances.
- The containerd@mgmt.server creates second instance of containerd over mgmt cgroup.
- The dockerd@mgmt.service creates the seconds of dockerd over cgroup and connects to second containerd instance through unix socket. 
- Communication between the between @mgmt services are through mgmt named unix socket.
- These servies are removed automatically when mgmt VRF is removed.

## tpcm@default Service (SONIC 4.1)
The TPCM@default service is a SONiC systemd service and it gets launched automatically on bootup.
- This service takes care of starting the dockerd@default.service and containerd@default.service instances. 
- The @default.services launch the third instance of dockerd and containerd instance under default category.  
- /var/lib/docker folder's default TPC services are to be placed under /host/tpcmdisk/docker-default directory during upgrade from older release to 4.1

## Resource Limitation and Configuration 
### Disk Storage (SONIC 4.1)
- TPCM feature will be supported on systems with a minimum of 32G disk space.
  Though the disk space is 32G, deducting the partition management overhead size, only around 29G will be usable. Hence we support disk space from 29G (29000M).
- Default TPC disk limit is set to 20% of overall disk space.( say, 20% of 32G disk = ~6.2G)
- Individual TPC disk limits are not supported.
- Automatic imposing of disk limits for TPCs on 4.1 image when upgraded from earlier releases. 
  However, if the disk utilization of the TPCs in earlier releases is greater than default overall TPC disk limit, 
  adjusted disk limit will be automatically set in order to support the TPCs running in older releases.
  To support the non-disruptive upgrade with respect to TPC,
  if the cumulative TPC disk usage in previous release exceeds the default TPC disk limit, then highest value plus a buffer disk size of 1G will be set as the new TPC disk limit. Buffer is provided to allow TPCs to work and expand in current release.
  For instance:
  7G is the previous release TPC disk size and the default 20% TPC disk limit is 6.2G on a 32G disk.
  During upgrade, as the required TPC disk size exceeds the default TPC limit size, 7G + 1G = 8G will be set as the new TPC disk limit to honor the existing TPCs.
  Also, if the allocation fails, then the underlying TPC framework (docker and container services) will not launch to support TPC.
  INFO Syslog will be generated to alert the user that TPC disk space is adjusted.

  Jul 01 05:59:22.346093+00:00 2022 sonic INFO tpcm.sh[5999]: [ tpcm ] Disk limit adjusted to 8322M

- Removal of TPC disk limits while migrating from current 4.1 release to older releases
- Updating the TPC disk limit is supported through CLI. 
- A new click as well as klish CLI is provided for this disk limit update(expand or shrink).
- A separate host disk image file system is created for TPCM and mounted.
- TPCs gets downloaded to /host/tpcmdisk (mounted host files system)
- Default TPCs to get downloaded to /host/tpcmdisk/docker-default and mgmt TPCs to get downloaded to /host/tpcmdisk/docker-mgmt
- Though the limit is set to 20% for the /host/tpcmdisk, it will not hard reserve/occupy the underlying base /host disk space. 
  It just allocates blocks and marks them as uninitialized.
  This allows the sonic owned dockers or the host apps to use the /host space to the fullest.
- Update in expansion of TPC disk limit checks for the available disk space in /host minus 1G buffer for sonic apps
- Update in shrink of TPC disk limit checks for the minimum file system blocks internally by resize2fs utility. 

click/Klish CLI to update tpcm disk limit:
    
    tpcm update disk-limit <value>

    <value> should be unit with one of the postfix `G` `M` `K` `g` `m` `k` characters
    <value> shall not be decimal
 

Instances on a 32G disk system:

- Default tpcm disk image and mounted disk size on a 32G disk system:
  The disk image file system is present in /host/disk-img/tpcm.ext4. 
  The tpcmdisk size could be measured from this file system as it is the logical size and
  the actual usable size will be shown in the mount point.

        root@sonic:/home/admin# ls -lh /host/disk-img/tpcm.ext4
        -rw-r--r-- 1 root root 6.3G Jul 18 15:14 /host/disk-img/tpcm.ext4
        root@sonic:/home/admin#
        root@sonic:/home/admin# df -h | grep tpcmdisk
        /dev/loop3      6.2G  248M  5.6G   5% /host/tpcmdisk

- klish tpcm update:
        
        sonic# tpcm update disk-limit 10G
        [ SUCCESS ] Update complete
        root@sonic:/home/admin# df -h | grep tpcmdisk
        /dev/loop2      9.8G  248M  9.1G   3% /host/tpcmdisk

        sonic# tpcm update disk-limit 5G
        [ SUCCESS ] Update complete
        root@sonic:/home/admin# df -h | grep tpcmdisk
        /dev/loop2      4.9G  245M  4.4G   6% /host/tpcmdisk

- Error Checks

        sonic# tpcm update disk-limit 500h
        %Error: [ ERROR ] Invalid input, The postfix should be one of the `G` `M` `K` `g` `m` `k` characters
        [ FAILED ] Update failed


        sonic# tpcm update disk-limit 10.5G
        %Error: [ ERROR ] Invalid input, Decimal value is not supported
        [ FAILED ] Update failed

        sonic# tpcm update disk-limit 500G
        %Error: [ ERROR ] disk-limit input is greater than current maximum allowed disk space(19230M or 18.78G)
        [ FAILED ] Update failed

        sonic# tpcm update disk-limit 10M
        %Error: [ ERROR ] disk-limit input is smaller than current minimum allowed disk limit(353M or 0.35G)
        [ FAILED ] Update failed


Installation on system with disk space less than 32G:
        
        sonic# tpcm install name 123 pull httpd
        New TPC docker image will be installed, continue? [y/N]: y
        [ ERROR ] TPC not supported in systems with disk space less than 32G
        [ FAILED ] Installation failed



### CPU Resource
- By default, for each TPC docker, the CPU resource limit is restricted to 20%.

### Memory Resource
- TPC overall memory limit is restricted to 20% of system memory  ( say, 3087MB from 15G system ram)
- By default, if individual tpc memory limit is not specified with the newly introduced args input(--memory) to "tpcm install",
  20% of TPC overall memory limit is assigned ( say, 617MB from 3087MB in a 15G system ram)
- Idea is that summation of the memory limits configured for all TPCs must not go beyond the overall TPC memory limit set.
- The value for --memory is designed to be parsed using docker.utils.parse_bytes which defines the postfix to the unit specified should be
  one of the `b` `k` `m` `g` characters (or) KB/MB/GB  or  K/M/G  or the unit without any postfix be consider a simple byte value.
- Also, --memory value shall not be less than 6MB as per the generic docker install criteria.
- Once the installation is done, the memory limit could be viewed from "docker inspect <TPC name>" or "docker stats" command.

click CLI args to support memory limit:
	
	tpcm install name <container_name> pull/file/image/scp/url/sftp..  < >   --args="--memory=<value>"

klish CLI args to support memory limit:
	
	tpcm install name <container_name> pull httpd args "--memory=<value"


Instances on a 15G RAM system:
		
- Default memory limit for each TPC will be set when the arg --memory is not issued:

		tpcm install name va1 pull httpd
		New TPC docker image will be installed, continue? [y/N]: y
		Memory limit 617MB set for va1 from overall TPC limit 3087MB
		Pulling the TPC-va1 image.
		Installing the TPC-va1 service file
		Installing the TPC-va1 config file
		Auto starting the TPC-va1
		[ SUCCESS ] Installation complete


- memory specified:

		tpcm install name va2 pull httpd --args="--memory=50m --net=host --privileged"
		New TPC docker image will be installed, continue? [y/N]: y
		Memory limit set for va2 is 50MB from overall TPC limit 3087MB
		Pulling the TPC-va2 image.
		Installing the TPC-va2 service file
		Installing the TPC-va2 config file
		Auto starting the TPC-va2
		[ SUCCESS ] Installation complete


- Error checks:

		tpcm install name va2 pull httpd --args="--memory=5h --net=host --privileged"
		New TPC docker image will be installed, continue? [y/N]: y
		The specified value for memory (5h) should specify the units. The postfix should be one of the `b` `k` `m` `g` characters
		[ FAILED ] Installation failed


		tpcm install name va2 pull httpd --args="--memory=5m --net=host --privileged"
		New TPC docker image will be installed, continue? [y/N]: y
		[ ERROR ] Minimum memory limit allowed is 6MB
		[ FAILED ] Installation failed

	    tpcm install name try7 pull httpd  --args="--memory=500m"
		New TPC docker image will be installed, continue? [y/N]: y
		[ ERROR ] Overall TPC memory limit reached- 3087MB   
		[ FAILED ] Installation failed



The same applies to "tpcm upgrade" CLI as well.

		tpcm upgrade name test1 pull httpd --args="--memory=1G"
		TPC docker will be upgraded to new image, continue? [y/N]: y
		Memory limit 1024MB for test1 set from overall TPC limit 3087MB
		Running PRE scripts for TPC-test1 image upgrade
		Removing the existing TPC image httpd
		Installing new TPC-test1 image
		Pulling the TPC-test1 image.
		Installing the TPC-test1 service file
		Installing the TPC-test1 config file
		Auto starting the TPC-test1
		Running POST scripts for TPC-test1 image upgrade
		[ SUCCESS ] Upgrade complete




## TPCM CLIs

- TPCM related operations are implemented through a CLI called 'tpcm'. Set of options with optional parameters shall be provided to perform TPCM operations. 

- Supported operations:
		
		tpcm install   => Install the TPC image into SONiC system.
		tpcm uninstall => Uninstall the TPC image from SONiC system.
		tpcm upgrade   => Upgrade the existing TPC container.
		tpcm list      => list the existing TPC images.
		tpcm update    => update the existing TPC container config


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

## TPC Update

- The TPCM framework allows the user to update the configs of TPC Dockers present already.
- The configs include the memory limit, the vrf and the start-after-system-ready.
- The flag --memory is used to update the memory limit of the TPC docker. This will not restart the TPC docker.
- The flag --vrf-name is used to update the vrf of the TPC docker. This will restart the TPC docker.
- The flag --start-after-system-ready is used to update the TPC service boot order to be after system ready or not.
- memory value is parsed by docker.utils.parse_bytes which defines the postfix to the unit specified should be one of the `b` `k` `m` `g` characters (or) KB/MB/GB  or  K/M/G  or the unit without any postfix be consider a simple byte value.
- vrf-name value is the string.
- All the flags or one of the flags could be specified to updated the TPC docker accordingly.

    - Example:

                tpcm update name <tpc-name>  --memory=<value> --vrf-name=<value> --start-after-ready=<True/False>


# CLI:

## 1. Installing the TPC image

TPC image can be loaded and installed in three forms:

#### TPC image from a file:
- It uses the download protocol to download the TPC image into a temporary path and then that gets loaded into Docker image filesystem. 
1. Load the TPC image file from a external http/https server

    - Syntax:
	    - #### tpcm install name \<tpc-container-name\> url \<URL\> [--vrf-name\<vrf-name\>] [--args \<docker args\>] [--cargs \<container args\>] [--start-after-system-ready\<False/True\>] [-y] 
	- where:
	    -  \<tpc-container-name\>  => Name of the TPC container to be loaded.
	    - \<URL\> => It is http/https web url from which the TPC image can be downloaded.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
		- --start-after-system-ready => It specifies if TPC to start after system ready or not.
        - [-y]  => User confirmation

	- Example:
    
	        tpcm install name mydocker url http://myserver/path/mydocker.tar.gz -y

2. Load the image file from a external server through scp/sftp

    - Syntax:
        - #### tpcm install name \<tpc-container-name> scp  \<server name>  --username \<username> [--password \<password>] --filename \<TPC image path> [--vrf-name\<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [--start-after-system-ready\<False/True\>] [-y] 
        - #### tpcm install name \<tpc-container-name> sftp \<server name>  --username \<username> [--password \<password>] --filename \<TPC image path> [--vrf-name\<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [--start-after-system-ready\<False/True\>] [-y] 
        
    - where:
        - \<tpc-container-name>  => Name of the TPC container to be loaded.
        - \<server name>    => Specifies the remote server name from which image is being download.
	    - --username  => Specifies the remote server user credentials.
	    - --password  => Specifies the remote server user's password. If passwd is not given, prompted passwd for during the image download.
	    - --filename  => Path on the remote server where the TPC image is located.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
		- --start-after-system-ready => It specifies if TPC to start after system ready or not.
	    - [-y] => User confirmation

    - Example:

            tpcm install name mydocker scp myserver  --username myuser --filename /path/mydocker.tar.gz -y
            Password:

3. Load the image file from a local media path

    - Syntax:
        - #### tpcm install name \<tpc-container-name> file  \<TPC image path> [--vrf-name\<vrf-name\>] [--args \<docker args>] [--cargs \<container args>] [--start-after-system-ready\<False/True\>] [-y]

    - where:
	     - \<TPC image path> => Local media path where the TPC image is located.
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
		- --start-after-system-ready => It specifies if TPC to start after system ready or not.
        - [-y]   => User confirmation

	- Example:

			tpcm install name mydocker file /media/usb/path/mydocker.tar.gz -y

		
4. Load the TPC image from a external Docker repo

    - It loads the TPC image directly from external Docker registry into local Docker image filesystem.

	- Syntax:
	    - #### tpcm install name \<tpc-container-name> pull \<Image name>[:\<tagname>] [--vrf-name\<vrf-name>] [--args \<docker args>] [--cargs \<container args>] [--start-after-system-ready\<False/True\>] [-y]

	- where 
	    - <tpc-container-name>  => Name of the TPC container to be loaded.
	    - <Image name>  => Name of the image to be downloaded.
	    - [tagname]  => Optional, Tagname of the image, by default 'latest' would be taken. 
        - \<vrf-name> => VRF name
	    - --args => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
		- --start-after-system-ready => It specifies if TPC to start after system ready or not.
        - [-y] => User confirmation.

    - Example:

			tpcm install name mydocker pull ubuntu:latest -y

5. Use one of the existing docker image

	- It uses one of the exising docker images installed in the system. It simply creates the service file and takes care of TPC and migration feature across the image  upgrade. This allows the user to use the regular docker commands to create the TPC images. Once the image is ready, user can associate TPCM management feature to it. 

	- Syntax:
	    - #### tpcm install name \<tpc-container-name> image \<Image name>[:\<tagname>] [--vrf-name\<vrf-name>] [--args <docker args>] [--cargs <container args>] [--start-after-system-ready\<False/True\>] [-y]

	- where 
	    - \<tpc-container-name>  => Name of the TPC container to be created.
	    - \<Image name>  => Name of the docker image in the local system.
	    - tagname  => Optional, Tagname of the image, by default 'latest' would be taken. 
        - \<vrf-name> => VRF name
	    - --dargs => It specifies the additional docker parameters.
	    - --cargs => It specifies the additional arguments to container init process or scripts.
		- --start-after-system-ready => It specifies if TPC to start after system ready or not.
        - [-y] => User confirmation
        
    - Example:

			tpcm install name mydocker image ubuntu:latest -y
	
## 2. Uninstalling the TPC image

- Uninstalling the tpc image

    - Syntax
        - #### tpcm uninstall name \<container-name\> [-y] [--clean_data]

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
        - #### tpcm list [--vrf-name\<vrf-name>]
    - Where
        - \<vrf-name> => VRF name

- Example
            
		#  tpcm list
		CONTAINER NAME  IMAGE TAG         VRF RUNNING/CONFIGURED            STATUS
		TEST            mydocker:latest   default/default                   Up 8 seconds

## 5.  Update the TPC container

- Update the TPC container that is installed on the system
    - Syntax
        - #### tpcm update [name|disk-limit <disk-limit-value> ] \<container-name\> [--memory\<mem-value>] [--vrf-name\<vrf-name>] [--start-after-system-ready\<False/True>]
    - Where
        - \<mem-value> => memory value
        - \<vrf-name> => VRF name
        - \<disk-limit-value> => overall tpcm disk limit value

- Example
            
		#  tpcm update name mydocker --memory=400MB --vrf-name=mgmt --start-after-system-ready=True
        #  tpcm update disk-limit 10G

## 6. TPC Service startup dependency
	
- By default, TPC will start only after the system(core services) is ready in 3.7
- If a user wants to make a TPC to start way early for a reason (say, to monitor memory in the boot up etc),
 "--start-after-system-ready=False" could be issued in the "tpcm install" command. But the user must be aware of the fact that certain functionalities may not work as excepted. For instance, host network(--network=host) wont be available to the TPC if it starts prior to hostcfgd. In such case, user has to explicitly update the service file accordingly.
		
		tpcm install name <tpcname> pull httpd --start-after-system-ready=<False/True>
	In install command, if this option is not specified, TPC will be configured to start after system ready by default
		
- "tpcm update" command will also have the "--start-after-system-ready" option to start TPC after the system is ready or not.
		
		tpcm update name <tpcname> --start-after-system-ready=<False/True>
		
- TPC migration from 3.4 to 3.7 sonic image: TPC services will be automatically updated to start after system is ready.
- TPC migration from 3.7 to 3.4 sonic image: TPC services' start after system ready boot order dependency will be removed
- TPC migration from 3.7 to >=3.7 sonic image: TPC services' configured boot dependency will be preserved.
		
- For TPC to start after a specific sonic service, following guidance will be provided in SONiC User Guide as well.

	Say for HTTPD1 TPC service to start after bgp service, 
		
		"sudo systemctl edit HTTPD1.service" 
	should be issued which opens up an editor in override.conf filename
	where the following could be added and saved
	
		[Unit]
		After=bgp.service
		



# KLISH CLI:

## 1. Installing the TPC image

#### TPC image from a file:

1. Load the TPC image file from a external http/https server

    - Syntax:
        #### tpcm install name \<tpc-container-name> url \<URL> [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]

        - Example:
            - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz
            - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz args " -e TESTENV=TESTVALUE"

2. Load the image file from a external server through scp/sftp

    - Syntax:
         - #### tpcm install name \<tpc-container-name> scp \<server name>  username \<username> password \<password> filename \<TPC image path> [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]
        - #### tpcm install name \<tpc-container-name> sftp <server name>  username \<username> password \<password> filename \<TPC image path> [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]

    - Example:

        - sonic# tpcm install name mydocker scp myserver  username myuser password paswd filename /path/mydocker.tar.gz

3. Load the image file from a local media path

- Load from the local file system
    - Syntax:
        - #### tpcm install name \<tpc-container-name> file  \<TPC image path> [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]

    - Example:
        - sonic# tpcm install name mydocker file /media/usb/path/mydocker.tar.gz
        - sonic# tpcm install name mydocker url http://myserver/path/mydocker.tar.gz args " -e TESTENV=TESTVALUE"


4. Load the TPC image from a external Docker repo

- Load from the docker hub.
    - Syntax:
        - #### tpcm install name \<tpc-container-name> pull \<Image name>[:\<Tagname>] [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]

    - Example:

            sonic# tpcm install name mydocker pull ubuntu:latest

5. Use one of the existing docker image

    - Syntax:
        - #### tpcm install name \<tpc-container-name> image \<Image name>[:\<Tagname>] [vrf-name\<vrf-name>] [args \<docker args>] [cargs \<container args>] [start-after-system-ready\<True/False>]
        
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

#### tpcm uninstall name \<container-name\> [vrf-name\<vrf-name>] [clean_data \<yes/no>]

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

#### show tpcm list [vrf-name\<vrf-name>] 
        - The TPCs can be listed using the tpcm list command.

        - Example:
           sonic#   show tpcm list
                    CONTAINER NAME  IMAGE TAG         VRF CONFIGURED/RUNNING            STATUS
                    TEST            mydocker:latest   default/default                   Up 8 seconds

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
		    - export DOCKER_HOST=unix:///run/docker-mgmt.socket
            - docker ps
	    - Use -H option to pass the socket option.
            - docker -H unix:///run/docker-mgmt.socket ps 


## 5.  Update the TPC images

#### tpcm update [name|disk-limit <disk-limit-value>] \<container-name\> [memory\<mem-value>] [vrf-name\<vrf-name>] [start-after-system-ready\<True/False>]
        - The TPCs can be updated using the tpcm update command.
        - Also, overall TPC disk limit can be updated.

        - Example:
           sonic#   tpcm update name TEST memory 200M vrf-name "mgmt" start-after-system-ready False
           sonic#   tpcm update disk-limit 10G

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
    28| Verify the TPC update of memory, disk.

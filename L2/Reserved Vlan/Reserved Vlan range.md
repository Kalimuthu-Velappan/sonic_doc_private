# Reserved VLAN range in SONiC

# High Level Design Document
#### Rev 0.3
# Table of Contents
* [List of Tables](#list-of-tables)
* [List of Figures](#list-of-figures)
* [Revision](#revision)
* [About this Manual](#about-this-manual)
* [Scope](#scope)
* [Definitions/Abbreviation](#definitionsabbreviation)
* [1 Overview](#1-overview)
    - [1.1 Use Cases](#11-use-cases)
* [2 Requirements](#2-requirements)
    - [2.1 Functional Requirements](#21-functional-requirements)
    - [2.2 Configuration and Management Requirements](#22-configuration-and-management-requirements)
* [3 Design](#3-design)
    - [3.1 CLI (and usage example)](#31-cli-and-usage-example)
        - [3.1.1 Configure Reserved Vlan range](#311-configure-reserved-vlan-range)
    - [3.2 Config DB](#32-config-db)
        - [3.2.1 RESERVED_VLAN_TABLE Table](#321-reserved-vlan-table)
    - [3.3 State DB](#33-state-db)
        - [3.3.1 RESERVED_VLAN_STATE_TABLE Table](#331-reserved-vlan-state-table)
    - [3.4 Notification](#34-notifications)
        - [3.4.1 RESERVEDVLANCHANGED](#341-reservedvlanchange)
        - [3.4.2 RESERVEDVLANALLOCATE](#341-reservedvlanallocate)
        - [3.4.3 RESERVEDVLANALLOCATED](#341-reservedvlanallocated)
    - [3.5 SWSS](#35-swss)
        - [3.5.1 Vlan Manager](#351-vlan-manager)
    - [3.6 KLISH CLI](#36-klish-cli)
        - [3.6.1 Show Commands](#361-show-commands)
        - [3.6.2 Config Commands](#362-config-commands)
    - [3.7 YANG model](#37-sonic-yang-model)
        - [3.7.1 Openconfig Yang model ](#371-openconfig-yang-model)
        - [3.7.2 Sonic Yang model ](#372-sonic-yang-model)
* [4 Unit Tests](#4-unit-tests)
* [5 Upgrade Scenario](#5-upgrade-scenario)
# List of Tables
* [Table 1: Abbreviations](#definitionsabbreviation)

# List of Figures

# Revision
| Rev | Date     | Author      | Change Description        |
|:---:|:--------:|:-----------:|---------------------------|
| 0.1 | 10/20/21 | Anil Pandey | Initial version           |
| 0.2 | 11/08/21 | Priyanka Gupta | Updated KLISH UI, yang models and UT section |
| 0.3 | 11/19/21 | Priyanka Gupta, Kamlesh Agrawal | Incorporated Review comments |

# About this Manual
This document provides an overview of the implementation of Reserved VLAN's in SONiC.

# Scope
This document describes the high level design of the Reserved VLAN's feature.

# Definitions/Abbreviation
| Abbreviation | Description                 |
|--------------|-----------------------------|
| PAC          | Port Access Control         |

## 1 Overview
The main goal of this feature is to provide a set of configurable VLAN's that are reserved for use by various protocols. 
- A set of Vlans will be reserved by default.
- A config CLI command will be provided to change the reserved Vlan range.

## 1.1 Use Cases


# 2 Requirements


## 2.1 Functional Requirements

- When a feature needing reserved-vlan is enabled, it will try and pick up a not-in-use vlan from the default range. If not available user must
	- Free up a vlan from that range OR
	- Change the reserved vlan range 
	- Until either of the above is setup, the feature will not function.

- When a feature needing reserved vlan is enabled and it finds a vlan in the range, it is responsible for creating that vlan using the internal VLAN manager provided interface. The feature is responsible for create/delete/modifies port-membership for that vlan. 

- Default reserved VLAN range would be <3967-4094>.

- All show commands will necessarily inform the user of any vlan usage within the current reserved vlan range and urge to move the use to a different vlan.

- There will also be warnings and syslog message (if a vlan in the reserved range is in use) that indicate that upon a migration to a future release, all user created config within the reserved vlan range will be deleted as part of migration. 

- User creation of vlans in reserved range will be blocked. But if a vlan is in use already (as part of migrated config), additional config on that vlan will be allowed.


## 2.2 Configuration and Management Requirements
This feature shall support CLI REST and GNMI based configurations.

List of configuration shall include the following:
- VLAN Id from where to start reserving VLANs



# 3 Design

## 3.1 CLI (and usage example)

### 3.1.1 Configure Reserved Vlan range

## 3.2 Config DB

One new table will be added to Config DB:
* RESERVED_VLAN_TABLE to store configured reserved vlan range

### 3.2.1 RESERVED_VLAN_TABLE Table

    ;Stores Reserved Vlan configuration
    key             = RESERVED_VLAN|"Vlan name"  


## 3.3 State DB
State DB will store information about:
* Reserved Vlans and whether it is in use


### 3.3.1 RESERVED_VLAN_TABLE Table

    ;Stores Reserved Vlan information
    key             = RESERVED_VLAN|"Vlan name" 
    inuse           = "true"/"false"  ; whether the reserved vlan is in use


## 3.4 Notifications
Notification will be sent from Vlanmgr to the consumers (Ex. PAC), indicating a change in reserved vlan range.


### 3.4.1 RESERVEDVLANCHANGED
Sent from Vlan Manager to indicate a change in Reserved Vlan Range.

    OP: ""
    DATA: ""
    VALUES: ""

### 3.4.2 RESERVEDVLANALLOCATE
Sent from consumers to Vlan Manager to request for a new Reserved Vlan

    OP: "SET/DEL"
    DATA: vlan_name
    VALUES: "consumer", consumer_name

### 3.4.3 RESERVEDVLANALLOCATED
Sent from Vlan Manager to consumer with a new reserved Vlan allocated

    OP: "SET"
    DATA: vlan_name
    VALUES: "consumer", consumer_name


## 3.5 SWSS

### 3.5.1 Vlan Manager

During boot up, VLAN Manager will update STATE_DB with default (or configured) reserved VLAN range. It will also update the 'in use' flag for each VLAN if the VLAN ID in reserved-vlan range is configured by the user.

When the Reserved Vlan range is changed from config, Vlan Manager will be notified through config db. Vlan Manager will then change the Reserved Vlan range in state db and also update the 'in use' flag for the Vlans already in use. 

Vlan Manager will also notify the consumers (Ex. PAC), indicating that there is a change in Reserved Vlan range. The consumers will then send request to Vlan Manager to allocate a new Vlan for its use. The cosumer will need to send the request to Vlan Manager and then wait for the response with new allocated reserved Vlan. The response from Vlan Manager will contain the consumer name and consumers should process response with their own consumer name and discard if otherwise.

The consumers will send a request to Vlan Manager to de-allocate a reserved Vlan that it no longer needs.

Vlan Manager will set the 'in use' flag for the Vlan that it chooses to allocate for the consumer.

VLAN manager will have a thread running to log messages on syslog periodically if there is a VLAN configured by the user from the reserved VLAN range.

<img src="images/reserved-vlan.jpg" width="800" height="500">


## 3.6 KLISH CLI


### 3.6.1 Show Commands
switch(config)# show system vlan reserved

  system vlan reservation: 400-527
        

### 3.6.2 Config Commands
switch(config)# system vlan 400 reserve 

"Previous reserve vlan range 100-228. 
 Continue anyway? (y/n) [no] y
 
  Reserves 128 contiguous vlans
  
switch(config)# no system vlan 400 reserve

  This command would change the reserved vlan range to default.

## 3.7 YANG model
## 3.7.1 Openconfig Yang model

  grouping reserve-vlan-config {
  
    description
    
      "Grouping for reserved Vlans";
      
    leaf vlan-name {
    
      type string;
      
      description
      
        "Vlan name";
	
    }
    
  }

  grouping reserve-vlan-top {
  
    description
    
      "This group contains information about all the reserved vlans";

    container reserve-vlans {
    
      description
      
        "Enclosure for list of Vlans";

    container reserve-vlan {

        description
	
           "List of reserved Vlans";

        container config {
	
          description
	  
            "Reserve Vlan Config data";

          uses reserve-vlan-config;
	  
        }

        container state {
	
          config false;
	  
          description
	  
            "Vlan state";
	    
          uses reserve-vlan-config;
	  
        }
	
      }
      
    }
    
  }
  
  uses reserve-vlan-top;
  
}

## 3.7.2 Sonic Yang model
    container RESERVED_VLAN {
            list RESERVED_VLAN_LIST {
                key "vlan-name";

                leaf vlan-name {
                    type string;

                }
             }
        }


# 4 Unit Tests
  1) Configure reserved vlan range using "system vlan <vlan-id> reserve" command and check if RESERVED_VLAN table got updated with contiguous 128 vlans.
  2) Configure reserved vlan range using "system vlan <vlan-id> reserve" command and check the o/p for "show system vlan reserved".
  3) Unconfigure the reserved vlan using "no system vlan <vlan-id> reserve" command and check if default reserved vlans are being used using show command.
  4) Configure reserved vlan range using "system vlan <vlan-id> reserve" command. Try to configure vlan/change vlan membership from reserved vlan range and check if proper error      messages are being displayed.
  5) Migration/upgrade from feature unsupported release to feature supported release, having user vlans in default vlan range.
  6) Try no system vlan reserve with the vlan-id not matching the start of the range. 

# 5 Upgrade Scenario
System upgrades from a release which does not have "Reserved VLAN" feature to a release with "Reserved VLAN" feature:
- There is no user config in the default Reserved VLAN range.
	- No action here. Also, the user will not be able to create any new VLAN's from the Reserved VLAN range
- There is user config conflicting with default Reserved VLAN range.
	- There will be warnings and syslog message that indicate that upon a migration to a future release, all user created config within the Reserved VLAN range will be deleted as part of migration. Also, all show commands will inform the user of any vlan usage within the current reserved vlan range and urge the user to move the use to a different vlan.
	

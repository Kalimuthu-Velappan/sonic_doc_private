
# Layer Routing Interface Counters

# High Level Design Document
#### Rev 0.1

# Table of Contents
  * [List of Tables](#list-of-tables)
  * [Revision](#revision)
  * [About This Manual](#about-this-manual)
  * [Scope](#scope)
  * [Definition/Abbreviation](#definitionabbreviation)
  * [1 Requirement Overview](#1-requirement-overview)
     * [1.1 Functional Requirements](#11-functional-requirements)
     * [1.2 Configuration and Management Requirements](#12-configuration-and-management-requirements)
     * [1.3 Scalability Requirements](#13-scalability-requirements)
     * [1.4 Warm Boot Requirements](#14-warm-boot-requirements)
     * [1.5 Limitiations](#15-limitations)
  * [2 Functionality](#2-functionality)
     * [2.1 Target Deployment Use Cases](#21-target-deployment-use-cases)
     * [2.2 Functional Description](#22-functional-description)
  * [3 Design](#3-design)
     * [3.1 Overview](#31-overview)
     * [3.2 DB Changes](#32-db-changes)
     * [3.3 Switch State Service Design](#33-switch-state-service-design)
     * [3.4 SyncD](#34-syncd)
     * [3.5 SAI](#35-sai)
     * [3.6 CLI](#36-cli)
  * [4 Flow Diagrams](#4-flow-diagrams)
  * [6 Warm Boot Support](#6-warm-boot-support)
  * [7 Scalability](#7-scalability)
  * [8 Unit Test](#8-unit-test)

# List of Tables
[Table 1: Abbreviations](#table-1-abbreviations)

# Revision
| Rev  |    Date     |   Author     | Change Description |
| :--: | :---------: | :---------:  | ------------------ |
| 0.1  | 28-Apr-2021 | Dilip Kumar,Laveen | Initial version    |
|      |             |              |                    |

# About this Manual
This document provides general information about supporting Layer 3 Routing Interface counters (interface statistics).

# Scope
This document describes the high level description of Layer 3 RIF counter support in SONiC. 

# Definition/Abbreviation
### Table 1: Abbreviations
| **Term** | **Meaning**                                       |
| -------- | ------------------------------------------------- |
| BGP      | Border Gateway protocol                           |
| CLI      | Command Line Interface                            |
| IP       | Internet Protocol                                 |
| L3       | Layer 3                                           |
| KLISH    | Industry Standard Command Line Interface (IS CLI) |
| RIF      | Routing Interface                                 |
| VRF      | Virtual Routing and Forwarding                    |
| SVI      | Switch Virtual Interface                          |


# 1 Requirement Overview

Provide Layer 3 Routing Interface counter support in SONIC and SAI. Routing interface includes router port, L3 PO, VLAN/SVI, Subinterface. 

## 1.1 Functional Requirements

  - Count Layer 3 (IPv4/IPv6) traffic received on each Routing Interface in both RX and TX directions
  - Count traffic per router interface in Bytes per seconds and packets per seconds in both RX and TX directions
  - Periodically read RIF counters from HW and update in COUNTER DB 


## 1.2 Configuration and Management Requirements

  - Provide IS CLI to display RIF Counters 
  - Provide IS CLI to clear RIF statistics/counters 
  - Provide IS CLI to configure the interval between two concecutive read/fetch from HW

## 1.3 Scalability Requirements

  - Support counters/statistics for all routing interfaces supported in the system. TD3 supports 4K Layer-3 routing interfaces. Counters/statistics collection will be supported on all the 4K interfaces. 

## 1.4 Warm Boot Requirements

  - Warmboot will result in discontinuity of the interface counters. That is, after warmboot, the interface counters will be rest back to zero and will start from there.

## 1.5 Limitiations

  - RIF Counters are not supported on Loopback Interfaces
  - RIF Counters are not supported on TD4, TH4 platforms due to HW limitations / unavailability of counter resources
  - RIF Counters does not show error counts, it could be either packets that are valid but not forwarded or invalid packets 
  - RIF counters will not account for CPU generated Tx packets
  - RIF counters rx stats will not account for packets destined to CPU


# 2 Functionality

## 2.1 Target Deployment Use Cases

## 2.2 Functional Description

# 3 Design
## 3.1 Overview

## 3.2 DB Changes

### 3.2.1 CONFIG DB

### 3.2.2 APP DB
Not applicable

### 3.2.3 STATE DB
Not applicable

### 3.2.4 ASIC DB
Not applicable

### 3.2.5 COUNTER DB


### 3.2.6 Configuration save and restart
Not applicable



## 3.3 Switch State Service Design
### 3.3.1 Orchestration Agent

### 3.3.2 Other Process 

## 3.4 SyncD


## 3.5 SAI

    RX:
        packets           - SAI_ROUTER_INTERFACE_STAT_IN_PACKETS
        bytes             - SAI_ROUTER_INTERFACE_STAT_IN_OCTETS

    TX:
        packets           - SAI_ROUTER_INTERFACE_STAT_OUT_PACKETS
        bytes             - SAI_ROUTER_INTERFACE_STAT_OUT_OCTETS

## 3.6 CLI

### 3.6.1 Data Models

Below is high level hierarchy of RIF counters sonic yang model

                      +--ro in-octets?               uint64
                      +--ro in-pkts?                 uint64
                      +--ro in-octets-per-second?    decimal64
                      +--ro in-pkts-per-second?      decimal64
                      +--ro in-bits-per-second?      decimal64
                      +--ro in-utilization?          scommon:percentage
                      +--ro out-octets?              uint64
                      +--ro out-pkts?                uint64
                      +--ro out-octets-per-second?   decimal64
                      +--ro out-pkts-per-second?     decimal64
                      +--ro out-bits-per-second?     decimal64
                      +--ro out-utilization?         scommon:percentage


### 3.6.2 Configuration Commands

Below is the command for configuring interval between 2 consecutive read/fetch for Layer 3 routing interface counters

        counters rif interval <value>
	
Default value for interval is 5 second. The range for interval is 3-30 seconds.


### 3.6.3 Show Commands

Below is the show command for displaying Layer 3 routing interface counters

	show interface counters rif

Here is a sample output of the above command

 sonic# show interface counters rif
 ----------------------------------------------------------------------------------------------
 	Interface      RX_OK    RX_BPS    RX_PPS    RX_ERR    TX_OK    TX_BPS    TX_PPS    TX_ERR
 ----------------------------------------------------------------------------------------------
     Ethernet46      561      100        32       N/A     34678       364      941       N/A
     Ethernet96        0        0         0       N/A         0         0        0       N/A
       Po33.101      258       N/A        4       N/A       698       N/A      N/A       N/A
        Vlan200     5432       58       123       N/A         0         0        0       N/A
    PortChannel001     0        0         0       N/A      4852        68       12       N/A
 ----------------------------------------------------------------------------------------------

     

### 3.6.4 Clear Commands

Below is the command for clearing Layer 3 routing interface counters

        clear counters interface rif
	
	
### 3.6.5 Debug Commands
Not applicable

### 3.6.6 REST API Support
Show Command

    	curl -X POST "https://x.x.x.x/restconf/operations/openconfig-counters-ext:rif-counters" -H "accept: application/yang-data+json" -H "Authorization: Basic c2s0MDgxMTU6RXBsZXJub3YxOA==" -H "Content-Type: application/yang-data+json" -d "{\"openconfig-counters-ext:input\":{\"riface\":\"all\"}}" -k

Clear Command

    curl -X POST "https://x.x.x.x/restconf/operations/openconfig-interfaces-ext:clear-rif-counters" -H "accept: application/yang-data+json" -H "Authorization: Basic YWRtaW46YnJvYWRjb20=" -H "Content-Type: application/yang-data+json" -d "{\"openconfig-interfaces-ext:input\":{\"rif\":\"all\"}}" -k
	
Config Command

    curl -X PATCH "https://x.x.x.x/restconf/data/openconfig-counters-ext:rif-counters/rif-counter=time_value/config" -H "accept: */*" -H "Content-Type: application/yang-data+json" -d "{\"openconfig-counters-ext:config\":{\"rif-counter-interval\":time_value}}" -k

### 3.6.7 Click Commands
The click show command is as below

         show interfaces counters rif [OPTOINS] [interface_name]

 	 Show interface counters

	 Options:

  	 -p, --period TEXT
	 
The click clear command is as below

        sonic-clear rifcounters
	
The click configuration command is as below

       counterpoll rif_rates interval <seconds>
       
For details of click commands please refer community HLD (https://github.com/Azure/SONiC/blob/eeebbf6f8a32e5d989595b0816849e8ef0d15141/doc/rif-counters/RIF_counters.md)

# 4 Flow Diagrams
Not applicable

# 5 Serviceability and Debug
Not applicable

# 6 Warm Boot Support

# 7 Scalability
Not applicable
# 8 Unit Test

1>    Test L3 forwarding for set of selected routes (IPV4 default vrf) and verify rif counters

2>    Clear rif counters with traffic paused (selected IPV4 routes in default vrf) and verify. Resume traffic and verify rif counters again.

3>    Configure interval to 60 seconds and verify rif counters for traffic (selected IPV4 routes in default vrf) before and after interval expires.

4>    Test L3 forwarding for set of selected routes (IPV4 non default vrf) and verify rif counters

5>    Test L3 forwarding for set of selected routes (IPV6 default vrf) and verify rif counters

6>    Test L3 forwarding for set of selected routes (IPV6 non default vrf) and verify rif counters

7>    Bidirectional traffic from orphan port of Peer1 to local MC-LAG port of Peer1 and verify rif counters.
 

# 9 Internal Design Information


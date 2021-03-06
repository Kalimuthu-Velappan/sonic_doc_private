# Feature Name
KLISH Debug Shell CLI framework
# High Level Design Document
#### Rev 0.6

# Revision

# About this Manual
This document provides general information about the executing [debugsh](#debugsh) CLI commands from sonic-cli/[KLISH](#KLISH).

# Scope
This document describes the high level design of providing infra structure for running  debugsh CLI  commands from sonic-cli. 

# Definition/Abbreviation

### Table 1: Abbreviations
| **Term** | **Meaning**                  |
| -------- | ---------------------------- |
| CLI      | Command Line Interface       |
| KLISH    | Kommand Line Interface Shell |
| debugsh  | SONiC Debug Shell            |

# Feature Overview

This feature provides the framework for integrating backend modules debug CLI commands with sonic-cli and the [Management Framework](#management-framework).

Currently debugsh supports 2 kinds of debug commands

* Backend or remote commands : Used to dump data stored in the heap memory of the backends. This information is typically not available in the redis DB. These commands will be supported in sonic-cli.

* Local or flex CLIs : These commands are used to gather information from various sources. For example backend, redis database, Linux commands, and the SDK shell. The information is then processed to show user friendly output like consistency checking, DB to Hardware mapping etc. These commands will not be supported.

**Only the debugsh backend commands are to be support in sonic-cli, and Management Framework.**

# Requirements

## Functional Requirements

Execute backend modules debugsh CLI commands from sonic-cli.

## Configuration and Management Requirements
N/A

## Scalability Requirements
N/A

## Warm Boot Requirements
N/A

# Design

## Overview

debugsh uses a PUB-SUB mechanism to communicate with the backends to build its CLI tree, and execute commands with the backends. In a similar fashion, the [Management Framework](#management-framework) debugsh app module, will implement a SONiC Yang RPC to communicate with the backends. The KLISH will use this RPC RESTConf operation to execute commands. However, the RPC callback will be unaware of the CLI tree, and it will leave that task to the debugsh infra. Please see the [CLI](#cli) subsection for more details.

Please see the [Flow Diagram](#flow-diagram).

## DB Changes
No planned DB Schema Changes

## User Interface

### Data Models

Outline of RPCs to be implemented

** sonic-debugsh.yang **

```

  rpc exec
    input: backend tokens
    (Eg: vxlanmgrd show system internal vxlanmgr global)

    output: list of strings
    (Eg:

        VxlanHwCapability Enable 
       Feature Capability: Vxlan(Enable) Vlan_Vni_Num(4096)

        ============== Pending Request Count===============
        tunnel_map:0 nvo:0 tunnel:0 vnet:0
    )

  rpc option
    input: optionName optionVal
    (Eg: timeout 2)

    output: {}

```

These SONiC RPCs are primarily for supporting the KLISH. Therefore, only the functionality needed for supporting KLISH is being implemented. Nothing prevents additional functionality from being added, say, for example, adding an RPC to get a listing of all the available debugsh CLI in the future.

### CLI

The debugsh infra already builds the CLI tree by parsing the CLI strings obtained from the backend. The debugsh CLI tree Klish XML file will be automatically generated by extending debugsh infra, and making this generated XML file available to the mgmt-framework container. A sanity check could be run on the generated XML file to ensure it is syntactically valid. The generation may take place right before the mgmt-framework container is declared up (i.e. when the RESTConf TCP port is available.). In normal circumstances, by the time networking infra is up, most backends should be ready to accept/process debugsh commands. If, however, a backend is not ready by then, another generation can take place right before the sonic-cli shell is launched for the missing backends. Currently, this operation is estimated to take in the order of 100ms. If greater time magnitudes are observed for this generation, an alternate background generation mechanism could be implemented.

#### Debug Commands

* **debug shell**     (To enter the debugsh view/mode)
* **show system internal ...**
* **debug system internal ...**
* **clear system internal ...**

* **timeout <1-180>** (Set the backend communication timeout)

** Due to the sensitive nature of these commands, they will be restricted to the admin user **


For example:

```
sonic-cli# debug shell
sonic-cli(debugsh)# show system internal <tab>
Possible completions:
aclsvcd       ACL Services daemon
coppmgr       CoppMgr daemon
debugsh       Debug Shell command
fdbsync       Fdbsync Services daemon
intfmgr       intfmgr related commands
nbrmgrd       nbrmgrd related commands
orchagent     Orchagent related commands
sagmgr        sagmgr related commands
sai           SAI Debug commands
stpmgr        stpmgr related commands
syncd         Syncd related commands
teammgr       TeamMgr related commands
vxlanmgr      vxlanmgr related commands
sonic-cli(debugsh)#
```

```
sonic-cli(debugsh)# show system internal vxlanmgr global 

 VxlanHwCapability Enable 
Feature Capability: Vxlan(Enable) Vlan_Vni_Num(4096)

 ============== Pending Request Count===============
 tunnel_map:0 nvo:0 tunnel:0 vnet:0
sonic-cli(debugsh)#
```

```
sonic-cli(debugsh)# debug system internal logger aclsvcd level info 
sonic-cli(debugsh)#
```

```
sonic-cli(debugsh)# clear system internal vlan ifp
sonic-cli(debugsh)# 
```

```
sonic-cli(debugsh)# timeout 2
sonic-cli(debugsh)#
```

#### IS-CLI Compliance
debugsh is a SONiC specific implementation, thus there is no need for IS-CLI compliance


### REST API Support
See [Data Models](#data-models)

### gNMI Support
No gNMI/gNOI access support is planned.

# Flow Diagrams

```

User  sonic-cli#       REST Server           Redis          Backend
____  __________       ___________           _____          _______

 o   (1)
-+-  CLI
 |  ----> .        
/ \       .     (2)       
          .     RPC       
          .     exec      
          . ------------> .
          .               .         (3)
          .               .       Publish
          .               . -----------------> .
          .               .                    .
          .               . --+                .
          .               .   |  (4)           .
          .               .   |  Wait          .
          .               .   |for o/p         .
          .               ..<-+                .
          .               ..                   .
          .               ..                   .      (5)
          .               ..                   .    Sub. Msg.
          .               ..                   . ------------> .
          .               ..                                   .
          .               ..                          (6)      .
          .               ..                        Publish    .
          .               ..                   . <------------ .
          .               ..                   .     o/p 1     .
          .               ..       (7)         .               .
          .               ..     Sub. Msg.     .               .
          .               ...<---------------- .               .
          .               ...     o/p 1                        .
          .               ...                                  .
          .     (8)       ...                                  .
          .    Chunk      ...                                  .
          . <------------ ...                                  .
          .    o/p 1      ..                                   .
          .               ..                                   .
     (9)  .               ..                                   .
    <---- .               ..                                   .
    o/p 1 .               ..                                   .
          .               ..                                   .
          .               ..                          (10)     .
          .               ..                        Publish    .
          .               ..                   . <------------ .
          .               ..                   .     o/p 2     .
          .               ..       (11)        .               .
          .               ..     Sub. Msg.     .               .
          .               ...<---------------- .               .
          .               ...     o/p 2                        .
          .               ...                                  .
          .     (12)      ...                                  .
          .    Chunk      ...                                  .
          . <------------ ...                                  .
          .    o/p 2      ..                                   .
          .               ..                                   .
     (13) .               ..                                   .
    <---- .               ..                                   .
    o/p 2 .               ..                                   .
          .               ..                                   .


                                 ...

          .               ..                                   .
          .               ..                         (n-3)     .
          .               ..                        Publish    .
          .               ..                   . <------------ .
          .               ..                   .     END
          .               ..      (n-2)        .
          .               ..     Sub. Msg.     .
          .               ...<---------------- .
          .               ...     END
          .               ...
          .    (n-1)      ...
          .    Chunk      ...
          . <------------ ...
          .
          .
     (n)  .
    <---- .


```

Notes:

- The o/p returned to the sonic-cli could be sent either buffered or in chunks
- For the initial release, we will implement returning all the data in one response chunk. Should a command which returns a large amount of output data be encountered, we could implement the chunked mechanism to prevent flow-control issues.

# Error Handling
N/A

# Serviceability and Debug
N/A

# Warm Boot Support
N/A

# Scalability
N/A

# Limitations

- Only the debugsh backend commands are to be supported in sonic-cli.
- If a developer adds more backend commands to a backend on a live system, they would have to regenerate the KLISH XML file for debugsh, by restarting the KLISH sonic-cli shell.
- If a backend takes a long amount of time to come online (relative to the mgmt-framework being declared up), the KLISH XML file would have to be regenerated, by restarting the KLISH sonic-cli shell, once that backend came online.

- KLISH sonic-cli handling of newlines is different from debugsh handing of newlines. Thus newlines (or other whitespace) differences between sonic-cli, and debugsh may exist. This behavior is also observed for KLISH sonic-cli save-to-file filter (i.e. a command similar to no-more which may appear after the pipe on the KLISH command line).

- If a backend does not send a newline after the last line, some versions of debugsh erase it while printing the prompt for the next command. This may be confirmed using typescript logs.

- Parameter descriptions, and detected syntax errors by the KLISH sonic-cli will follow KLISH style, and thus will be different from debugsh styles.

- Restriction checking for keyword prefixed integer ranges (Eg: Trigger[1-32]) is an approximation. No known examples exist so far, but the debugsh capability exists for this. A tighter restriction can be developed with more time investment, when an example shows up.


# Unit Test

| **Test**                       | **Description**                          |
| ------------------------------ | ---------------------------------------- |
|                                |                                          |
| KLISH XML Autogeneration       | Generate KLISH XML by running sonic-cli  |
|                                |                                          |
| REST Yang RPC (exec)           | SONiC debugsh RPC for executing backend  |
|                                | debugsh commands                         |
|                                |                                          |
| REST Yang RPC (option)         | SONiC debugsh RPC for setting options    |
|                                | for communicating with backend           |
|                                |                                          |
| KLISH show                     | Execute KLISH debugsh show command and   |
|                                | compare with native debugsh command      |
|                                |                                          |
| KLISH debug                    | Execute KLISH debugsh debug command and  |
|                                | compare with native debugsh command      |
|                                |                                          |
| KLISH clear                    | Execute KLISH debugsh clear command and  |
|                                | compare with native debugsh command      |
|                                |                                          |
| KLISH timeout                  | Execute KLISH debugsh timeout command and|
|                                | compare with native debugsh command      |
|                                |                                          |

# References

## Management Framework

https://github.com/Azure/SONiC/blob/master/doc/mgmt/Management%20Framework.md

## KLISH

https://github.com/Azure/SONiC/blob/master/doc/mgmt/Management%20Framework.md#3221-cli

## debugsh

https://docs.google.com/document/d/1kMpJ2k4CIBn-08TTKlYqxTBlDrmVNg5El7htcJ5sPxc

"Debugsh provides a mechanism to display backend data which is typically not available in the database. It can also be used to enable certain debug features during runtime on need basis like enabling some debug prints, debug checks etc."



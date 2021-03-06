#  SQA Test Plan
#  SONIC Interface Down Reason
#  SONiC 4.0.0 Release

## Test Plan Revision History

| Rev  | Date       | Author          | Change Description |
| ---- | ---------- | --------------- | ------------------ |
| 0.1  | 6/11/2021  | Jagadish Ch     | Initial Version    |
| 0.2  | 6/14/2021  | Jagadish Ch     | Addressed review comments provided by Prasanth Kunjum Vettil |
| 0.3  | 6/15/2021  | Jagadish Ch     | Addressed review comments provided by Naveen Nagaraju |
| 0.4  | 6/21/2021  | Jagadish Ch     | Addressed review comments provided by Madhukar Kamarapu |
| 0.5  | 7/17/2021  | Jagadish Ch     | Adding MC-LAG test case |

## List of Reviewers

| Function | Name |
| :------: | :--: |
|    QA      |   Chandra Bhushan Singh   |
|    DEV     |   Prasanth K V	    |

## List of Approvers

| Function | Name | Date Approved |
| :------: | :--: | :-----------: |
|          |      |               |
|          |      |               |

## Definition/Abbreviation

| **Term** | **Meaning**                |
| -------- | -------------------------- |
|          |                            |
|          |                            |

## Feature Overview

  This feature is to help the user in finding the reason for Physical/PortChannel interfaces down easier.

## 1 Test Focus Areas

  - This Test plan covers the validation of all the down reasons of Physical/PortChannel interfaces.

### 1.1 Functional Testing

  - Validate interface down reason "Admin down" for physical interface.
  - Validate interface down reason "Remote fault" for physical interface.
  - Validate interface down reason "Transceiver not present" for physical interface.
  - Validate interface down reason "Incompatible transceiver" for physical interface.
  - Validate interface up reason "PHY link up" for physical interface.
  - Validate interface down reason "Port breakout in-progress" for physical interface.
  - Validate interface down reason "STP error disabled" for physical interface with STP mode as "pvst".
  - Validate interface down reason "STP error disabled" for physical interface with STP mode as "rapid-pvst".
  - Validate interface down reason "UDLD error disabled" for physical interface.
  - Validate interface down reason "Link flap error disabled" for physical interface.
  - Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "pvst".
  - Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "rapid-pvst".
  - Validate interface down reason "Min-links-not-met" for PortChannel interface.
  - Validate interface down reason "Admin-down" for PortChannel interface.
  - Validate interface down reason "LACP-convergence-failed" for PortChannel interface.
  - Validate interface down reason "All-links-down" for PortChannel interface.
  - Validate interface down reason for the uplink tracking enabled port of MC-LAG peer when it's MC-LAG is down.
  - Below interface down reasons will be added later.<BR/>
  Link training failed<BR/>
  Link training not completed<BR/>
  Link training not started<BR/>
  Link tuning failed<BR/>
  Link tuning not started<BR/>
  Link tuning not completed<BR/>
  High BER<BR/>
  PCS AM lock error<BR/>
  PCS sync error<BR/>
  Transceiver error disabled<BR/>
  


## 2 Topologies

<span style="color:red"> </span>![Topology](Interface_Down_Reason.png "Figure: Topology 1")


#### 								Topology 1
Topology Description -
- Configure PortChannel7 in DUT1 and DUT2 and add inter connected ports as members of that PortChannel.<BR/>
**Note:**  This configuration is applicable only for the PortChannel test cases except test cases INTERFACE_DOWN_REASON_FUNC_015 and INTERFACE_DOWN_REASON_FUNC_016.

<span style="color:red"> </span>![Topology](Interface_Down_Reason2.png "Figure: Topology 2")


#### 								Topology 2
Topology Description -
- Configure PortChannel7 in DUT1, DUT2, DUT3 and PortChannel8 only in DUT2.<BR/>
- Configure Ethernet1, Ethernet2, Ethernet3, Ethernet4, Ethernet5 and Ethernet6 as members of PortChannel7 in DUT1.<BR/>
- Configure Ethernet1 and Ethernet2 as members of PortChannel8 in DUT2.<BR/>
- Configure Ethernet3 and Ethernet4 as members of PortChannel7 in DUT2.<BR/>
- Configure Ethernet5 and Ethernet6 as members of PortChannel7 in DUT3.<BR/>
**Note:**  This configuration is applicable only for the PortChannel test case INTERFACE_DOWN_REASON_FUNC_015.

<span style="color:red"> </span>![Topology](Interface_Down_Reason_Mclag.png "Figure: Topology 3")

#### Topology 3
**Note:**  This topology/configuration is applicable only for the MC-LAG test case INTERFACE_DOWN_REASON_FUNC_019.




## 3 Test Case and objectives


### 3.1 Functional Test Cases

#### 3.1.1 Validate interface down reason "Admin down" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_001**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Admin down" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Shutdown the Ethernet1 in DUT1 and observe the down reason for that port is updated as "Admin-down" in the "show interface status" output.|

#### 3.1.2 Validate interface down reason "Remote fault" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_002**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Remote fault" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Shutdown the interface Ethernet2 in DUT2.<BR/>2. Observe the down reason for that port is updated as "Admin down" in DUT2 and "Phy-link-down" in DUT1 for the corresponding port(Ethernet2) in the "show interface status" output.<br/>3. Observe "remote-fault" event with time stamp in "show interface Ethernet" output of DUT1.|

#### 3.1.3 Validate interface down reason "Transceiver not present" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_003**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Transceiver not present" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Don't insert any transceiver in the SFP/SFP+/QSFP/QSFP+ module of DUT1.<BR/>2. Observe interface down reason will be shown as "Phy-link-down" for the respective ports in the "show interface status" output of DUT1.<BR/>3. Observe "Transceiver not present" event in "show interface Ethernet" output of DUT1.|

#### 3.1.4 Validate interface down reason "Incompatible transceiver" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_004**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Incompatible transceiver" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Insert unsupported/speed incompatible transceivers in Ethernet3 of DUT1, supported transceiver in Ethernet3 of DUT2.<BR/>2. Observe the interface down reason for Ethernet3 is updated as "Phy-link-down" in "show interface status" output of DUT1.<BR/>3. Observe "Incompatible transceiver" event in "show interface Ethernet" output of DUT1.|

#### 3.1.5 Validate link up reason "PHY link up" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_005**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate link up reason "PHY link up" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Make the port as up using PHY loopback and observe the link up reason for the port is updated as "PHY link up" in "show interface status" output.|

#### 3.1.6 Validate interface down reason "Port breakout in-progress" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_006**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Port breakout in-progress" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Breakout the port to supported modes.<BR/>2. Observe the interface down reason for the port is updated as "Phy-link-down" during the breakout process in "show interface status" output.<BR/>3. Observe "Port breakout in-progress" event in "show interface Ethernet" output of DUT1.|

#### 3.1.7 Validate interface down reason "STP error disabled" for physical interface with STP mode as "pvst".  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_007**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "STP error disabled" for physical interface with STP mode as "pvst".** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure VLAN-10 in DUT1 and DUT2.<BR/>2. Add Ethernet1 as tagged member of VLAN-10 in DUT1 and DUT2.<BR/>3. Configure Spanning tree mode as "pvst" in DUT1 and DUT2.<BR/>4. Enable spanning on VLAN-10 in DUT1 and DUT2.<BR/>5. Enable spanning tree on Ethernet1 of DUT1 and DUT2.<BR/>6. Configure errdisable recovery cause bpduguard in DUT1.<BR/>7. Configure errdisable recovery interval as 30 seconds in DUT1.<BR/>8. Now enable bpdu guard on Ethernet1 of DUT1 with port shutdown on bpdu guard violation.<BR/>9. Observe Ethernet1 will go down and the interface down reason will be updated as "Err-disabled" in the "show interface status" output in DUT1.<BR/>10. Observe the event is displayed as "STP-down" for Ethernet1 in the "show interface status err-disabled" and "show interface Ethernet" outputs of DUT1.<BR/>11. Disable spanning tree on Ethernet1 of DUT2<BR/>12. Observe after the recovery interval(30 seconds) Ethernet1 will come Up in DUT1.|

#### 3.1.8 Validate interface down reason "STP error disabled" for physical interface with STP mode as "rapid-pvst".  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_008**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "STP error disabled" for physical interface with STP mode as "rapid-pvst".** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure VLAN-10 in DUT1 and DUT2.<BR/>2. Add Ethernet1 as tagged member of VLAN-10 in DUT1 and DUT2.<BR/>3. Configure Spanning tree mode as "rapid-pvst" in DUT1 and DUT2.<BR/>4. Enable spanning on VLAN-10 in DUT1 and DUT2.<BR/>5. Enable spanning tree on Ethernet1 of DUT1 and DUT2.<BR/>6. Configure errdisable recovery cause bpduguard in DUT1.<BR/>7. Configure errdisable recovery interval as 30 seconds in DUT1.<BR/>8. Now enable bpdu guard on Ethernet1 of DUT1 with port shutdown on bpdu guard violation.<BR/>9. Observe Ethernet1 will go down and the interface down reason will be updated as "Err-disabled" in the "show interface status" output in DUT1.<BR/>10. Observe the event is displayed as "STP-down" for Ethernet1 in the "show interface status err-disabled" and "show interface Ethernet" outputs of DUT1.<BR/>11. Disable spanning tree on Ethernet1 of DUT2<BR/>12. Observe after the recovery interval(30 seconds) Ethernet1 will come Up in DUT1.|

#### 3.1.9 Validate interface down reason "UDLD error disabled" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_009**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "UDLD error disabled" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure errdisable recovery cause udld in DUT1.<BR/>2. Configure errdisable recovery interval as 30 seconds in DUT1.<BR/>3. Enable UDLD globally in DUT1 and DUT2.<BR/>4. Configure message-time and multiplier as 2 and 3 respectively in DUT1 and DUT2.<BR/>5. Enable UDLD on Ethernet1 of both DUT1 and DUT2.<BR/>6. Enable UDLD packets blocking on Ethernet1 of DUT1<BR/>7. Observe Ethernet1 will go down and the interface down reason will be updated as "Err-disabled" in the "show interface status" output in DUT1.<BR/>8. Observe the event is displayed as "UDLD error disabled" for Ethernet1 in the "show interface status err-disabled" and "show interface Ethernet" outputs of DUT1.<BR/>9. Disable UDLD packets blocking on Ethernet1 of DUT1.<BR/>10. Observe after the recovery interval(30 seconds) Ethernet1 will come Up in DUT1.|

#### 3.1.10 Validate interface down reason "Link flap error disabled" for physical interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_010**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Link flap error disabled" for physical interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Enable link flap error disable on Ethenet4 of DUT1 with flap-threshold 2, sampling-time 10, recovery-timeout 15.<BR/>2. Do shutdown and no-shutdown Ethernet4 of DUT2 for 3 times within 10 seconds.<BR/>3. Observe the interface down reason for Ethernet4 will be updated as "Err-disabled" in the "show interface status" output of DUT1 .<BR/>4. Observe the event is displayed as "Link flap error disabled" for the interface Ethernet4 in the "show interface status err-disabled" and "show interface Ethernet" outputs of DUT1.<BR/>5. The Link flap error disabled interface will come up after 15 seconds of recovery-time.|

#### 3.1.11 Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "pvst".  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_011**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "pvst".** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure VLAN-10 in DUT1 and DUT2.<BR/>2. Add PortChannel7 as tagged member of VLAN-10 in DUT1 and DUT2.<BR/>3. Configure Spanning tree mode as "pvst" in DUT1 and DUT2.<BR/>4. Enable spanning on VLAN-10 in DUT1 and DUT2.<BR/>5. Enable spanning tree on PortChannel7 of DUT1 and DUT2.<BR/>6. Configure errdisable recovery cause bpduguard in DUT1.<BR/>7. Configure errdisable recovery interval as 30 seconds in DUT1.<BR/>8. Now enable bpdu guard on PortChannel7 of DUT1 with port shutdown on bpdu guard violation.<BR/>9. Observe PortChannel7 will go down and the interface down reason will be updated as "Err-disabled" in "show interface PortChannel" and "show interface status" outputs of DUT1.<BR/>10. Observe the event is displayed as "STP-down" for PortChannel7 in the "show interface status err-disabled" output of DUT1.<BR/>11. Disable spanning tree on PortChannel7 of DUT2<BR/>12. Observe after the recovery interval(30 seconds) PortChannel7 will come Up in DUT1.|

#### 3.1.12 Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "rapid-pvst".  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_012**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "STP error disabled" for PortChannel interface with STP mode as "rapid-pvst".** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure VLAN-10 in DUT1 and DUT2.<BR/>2. Add PortChannel7 as tagged member of VLAN-10 in DUT1 and DUT2.<BR/>3. Configure Spanning tree mode as "rapid-pvst" in DUT1 and DUT2.<BR/>4. Enable spanning on VLAN-10 in DUT1 and DUT2.<BR/>5. Enable spanning tree on PortChannel7 of DUT1 and DUT2.<BR/>6. Configure errdisable recovery cause bpduguard in DUT1.<BR/>7. Configure errdisable recovery interval as 30 seconds in DUT1.<BR/>8. Now enable bpdu guard on PortChannel7 of DUT1 with port shutdown on bpdu guard violation.<BR/>9. Observe PortChannel7 will go down and the interface down reason will be updated as "Err-disabled" in "show interface PortChannel" and "show interface status" outputs of DUT1.<BR/>10. Observe the event is displayed as "STP-down" for PortChannel7 in the "show interface status err-disabled" output of DUT1.<BR/>11. Disable spanning tree on PortChannel7 of DUT2<BR/>12. Observe after the recovery interval(30 seconds) PortChannel7 will come Up in DUT1.|


#### 3.1.13 Validate interface down reason "Min-links-not-met" for PortChannel interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_013**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Min-links-not-met" for PortChannel interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure the PortChannels with min-links as 4 in both the DUTs.<BR/>2. Remove one of the member ports from PortChannel in DUT1.<BR/>3. Observe that interface down reason will be updated as "Min-links" in "show interface status" and "show interface PortChannel" outputs of DUT1.<BR/>4. Add the member port back to the PortChannel and observe PortChannel is Up in both the devices.<BR/>5. Shutdown one of the member ports of PortChannel in DUT1.<BR/>6. Observe that interface down reason will be updated as "Min-links" in "show interface status" and "show interface PortChannel" outputs of DUT1.<BR/>7. no shutdown the member port of PortChannel in DUT1 and observe PortChannel is Up in both the devices.<BR/>8. Remove one of the member ports from PortChannel in DUT2.<BR/>9. Observe that interface down reason will be updated as "Min-links" in "show interface status" and "show interface PortChannel" outputs of DUT1.<BR/>10. Add the member port back to the PortChannel and observe PortChannel is Up in both the devices.<BR/>|

#### 3.1.14 Validate interface down reason "Admin-down" for PortChannel interface. 

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_014**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Admin-down" for PortChannel interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Shutdown the PortChannel in DUT1 and observe the down reason for that PortChannel is updated as "Admin down" in the "show interface PortChannel" and "show interface status" outputs of DUT1.<BR/>2. Do no-shutdown the PortChannel in DUT1 and shutdown the PortChannel in DUT2.<BR/>3. Observe the interface down reason is shown as "All-links-down" in "show interface PortChannel" and "show interface status" outputs of DUT1 and shown as "Admin-down" in DUT2.|

#### 3.1.15 Validate interface down reason "LACP-convergence-failed" for PortChannel interface.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_015**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "LACP-convergence-failed" for PortChannel interface.** |
| **Test Setup** | **Topology 2**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure PortChannel7 in DUT1, DUT2 and DUT3, PortChannel8 only in DUT2.<BR/>2. Add Ethernet1, Ethernet2, Ethernet3, Ethernet4, Ethernet5, Ethernet6 as members of PortChannel7 in DUT1.<BR/>3. Add Ethernet1, Ethernet2 as members of PortChannel8 in DUT2.<BR/> 4. Add Ethernet3, Ethernet4 as members of PortChannel7 in DUT2.<BR/>5. Add Ethernet5, Ethernet6 as members of PortChannel7 in DUT3.<BR/>6. Observe PortChannel7 is Up with Ethernet1 and Ethernet2 as the only active members in DUT1.<BR/>7. Observe the interface down reason for PortChannel7 is shown as "LACP-fail" in "show interface status" and "show interface PortChannel" outputs of DUT2 and DUT3.<BR/>**Note:** Strictly follow the sequence of above steps.|

#### 3.1.16 Validate interface down reason "LACP-convergence-failed" for PortChannel interface with reboot.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_016**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "LACP-convergence-failed" for PortChannel interface with reboot.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Configure PortChannel7 in DUT1, PortChannel7 and PortChannel8 in DUT2.<BR/>2. Add Ethernet1, Ethernet2, Ethernet3, Ethernet4 as members of PortChannel7 in DUT1.<BR/>3. Add Ethernet1, Ethernet2 as members of PortChannel8.<BR/> 4. Add Ethernet3, Ethernet4 as members of PortChannel7 in DUT2.<BR/>5. Observe PortChannel7 is Up with Ethernet1 and Ethernet2 as the only active members in DUT1.<BR/>6. Observe PortChannel8 is Up with Ethernet1 and Ethernet2 as the only active members in DUT2.<BR/>7. Observe the interface down reason for PortChannel7 is shown as "LACP-fail" in "show interface status" and "show interface PortChannel" outputs of DUT2.<BR/>8. Save the configuration and reboot DUT1 and DUT2.<BR/>9. After the devices Up PortChannel7 will come up with either Ethernet1, Ethernet2 or Ethernet3, Ethernet4 in DUT1.<BR/>10. Either PortChannel7 or PortChannel8 will Up in DUT2 depending on the port Up after the device up after reboot.<BR/>Here considering the member ports of PortChannel8 will come up first in DUT2.<BR/>11. Observe the interface down reason for PortChannel7 is shown as "LACP-fail" in "show interface status" and "show interface PortChannel" outputs of DUT2<BR/>**Note:** Strictly follow the sequence of above steps.|

#### 3.1.17 Validate interface down reason "Min-links-not-met" for PortChannel interface by pulling the cable manually.  

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_017**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "Min-links-not-met" for PortChannel interface by pulling the cable manually.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      |  1. Configure the PortChannel7 with min-links as 4 in DUT1 and DUT2.<BR/>2. Add Ethernet1, Ethernet2, Ethernet3, Ethernet4 as members of PortChannel7 in DUT1 and DUT2.<BR/>3. Observe the PortChannel7 is up with Ethernet1, Ethernet2, Ethernet3, Ethernet4 as active members in DUT1 and DUT2.<BR/>4. Plug out the cable connected to Ethernet1 of DUT2.<BR/>5. Observe that interface down reason will be updated as "Min-links" in "show interface status" and "show interface PortChannel" outputs of DUT1 and DUT2.<BR/>6. Plug in the cable back and observe PortChannel is Up in both the devices.|

#### 3.1.18 Validate interface down reason "All-links-down" for PortChannel interface. 

| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_018**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason "All-links-down" for PortChannel interface.** |
| **Test Setup** | **Topology 1**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Shutdown the PortChannel in DUT2.<BR/> 2. Observe the interface down reason for that PortChannel is updated as "Admin down" in the "show interface PortChannel" and "show interface status" outputs of DUT2.<BR/>3. Observe the interface down reason for PortChannel is updated as "All-links-down" in the "show interface PortChannel" and "show interface status" outputs of DUT1.<BR/>4. Do no-shutdown the PortChannel in DUT2.<BR/>5. Observe the PortChannel is Up in both DUT1 and DUT2.<BR/>6. Shutdown Ethernet1, Ethernet2, Ethernet3 and Ethernet4 in DUT2 which are members of PortChannel.<BR/>7. Observe the interface down reason for PortChannel is updated as "All-links-down" in the "show interface PortChannel" and "show interface status" outputs of DUT1.|

#### 3.1.19 Validate interface down reason for the uplink tracking enabled port of MC-LAG peer when it's MC-LAG is down.
| **Test ID**    | **INTERFACE_DOWN_REASON_FUNC_019**                                               |
| -------------- | :----------------------------------------------------------- |
| **Test Name**  | **Validate interface down reason for the uplink tracking enabled port of MC-LAG peer when it's MC-LAG is down.** |
| **Test Setup** | **Topology 3**                                               |
| **Type**       | **Functional**                                               |
| **Steps**      | 1. Bring up MCLAG as shown in Topology3.<BR/>2. Configure any link up port(Except LAG members)in Peer-1 as up-link port and enable up-link tracking feature on it.<BR/>3. Now make the MC-LAG as down in Peer-1.<BR/>4. Observe the up-link port is down due to the up-link tracking feature with proper reason code.<BR/>

<BR/>
<BR/>






### 3.4 Management

#### 3.4.1 REST

## Reference Links

https://github.com/BRCM-SONIC/sonic_doc_private/blob/master/system/Interface_Down_Reason.md
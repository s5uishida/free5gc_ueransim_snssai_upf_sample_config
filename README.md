# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses free5GC and UERANSIM to select the UPF based on S-NSSAI.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of free5GC 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of free5GC 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of free5GC 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of free5GC 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB)](#changes_ran)
    - [Changes in configuration files of UE[SST:1, SD:0x000001] (IMSI-001010000000000)](#changes_ue_sd1)
    - [Changes in configuration files of UE[SST:1, SD:0x000002] (IMSI-001010000000000)](#changes_ue_sd2)
- [Network settings of free5GC 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of free5GC 5GC C-Plane](#network_settings_cp)
  - [Network settings of free5GC 5GC U-Plane1](#network_settings_up1)
  - [Network settings of free5GC 5GC U-Plane2](#network_settings_up2)
- [Build free5GC and UERANSIM](#build)
- [Run free5GC 5GC and UERANSIM UE / RAN](#run)
  - [Run free5GC 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run free5GC 5GC C-Plane](#run_cp)
  - [Run UERANSIM (gNodeB)](#run_ran)
  - [Run UERANSIM (UE[SST:1, SD:0x000001])](#run_sd1)
    - [UE connects to U-Plane1 based on SST:1 and SD:0x000001](#con_sd1)
    - [Ping google.com going through DN=10.60.0.0/16 on U-Plane1](#ping_sd1)
  - [Run UERANSIM (UE[SST:1, SD:0x000002])](#run_sd2)
    - [UE connects to U-Plane2 based on SST:1 and SD:0x000002](#con_sd2)
    - [Ping google.com going through DN=10.61.0.0/16 on U-Plane2](#ping_sd2)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of free5GC 5GC Simulation Mobile Network

The following minimum configuration was set as a condition.
- The UE selects a pair of SMF and UPF based on S-NSSAI.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v3.4.1 (2024.03.28) - https://github.com/free5gc/free5gc
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | free5GC  5GC C-Plane | 192.168.0.141/24 <br> 192.168.0.142/24 <br> 192.168.0.143/24 | Ubuntu 22.04 | 2GB | 20GB |
| VM2 | free5GC  5GC U-Plane1  | 192.168.0.144/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | free5GC  5GC U-Plane2  | 192.168.0.145/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported S-NSSAI |
| --- | --- | --- | --- |
| AMF | 192.168.0.141 | 127.0.0.18 | SST:1, SD:0x000001 <br> SST:1, SD:0x000002 |
| SMF1 | 192.168.0.142 | 127.0.0.2 | SST:1, SD:0x000001 |
| SMF2 | 192.168.0.143 | 127.0.0.12 | SST:1, SD:0x000002 |

gNodeB Information (other information is default) is as follows.  
| IP address | Supported S-NSSAI |
| --- | --- |
| 192.168.0.131 | SST:1, SD:0x000001 <br> SST:1, SD:0x000002 |

Subscriber Information (other information is default) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE | IMSI | DNN | OP/OPc | S-NSSAI |
| --- | --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc | SST:1, SD:0x000001 <br> SST:1, SD:0x000002|

I registered these information with the free5GC WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | S-NSSAI |  TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- | --- |
| 10.60.0.0/16 | SST:1 <br> SD:0x000001 | upfgtp | internet | uesimtun0 | U-Plane1 |
| 10.61.0.0/16 | SST:1 <br> SD:0x000002 | upfgtp | internet | uesimtun0 | U-Plane2 |

<a id="changes"></a>

## Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

- `free5gc/config/amfcfg.yaml`
```diff
--- amfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ amfcfg.yaml 2024-03-30 10:48:03.417452740 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.141
   ngapPort: 38412 # the SCTP port listened by NGAP
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
@@ -24,23 +24,23 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
   supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
   plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+          sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+          sd: 000002 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
   supportDnnList:  # the DNN (Data Network Name) list supported by this AMF
     - internet
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2024-03-30 10:35:10.534612278 +0900
+++ ausfcfg.yaml        2024-03-30 10:48:53.470630936 +0900
@@ -16,10 +16,8 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-    - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
   eapAkaSupiImsiPrefix: false # including "imsi-" prefix or not when using the SUPI to do EAP-AKA' authentication
 
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ nrfcfg.yaml 2024-03-31 12:34:36.380715909 +0900
@@ -15,8 +15,8 @@
       key: cert/nrf.key # NRF TLS Private key
     oauth: true
   DefaultPlmnId:
-    mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-    mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+    mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   serviceNameList: # the SBI services provided by this NRF, refer to TS 29.510
     - nnrf-nfm # Nnrf_NFManagement service
     - nnrf-disc # Nnrf_NFDiscovery service
```
- `free5gc/config/nssfcfg.yaml`
```yaml
info:
  version: 1.0.2
  description: NSSF initial local configuration

configuration:
  nssfName: NSSF
  sbi:
    scheme: http
    registerIPv4: 127.0.0.31
    bindingIPv4: 127.0.0.31
    port: 8000
    tls:
      pem: cert/nssf.pem
      key: cert/nssf.key
  serviceNameList:
    - nnssf-nsselection
    - nnssf-nssaiavailability
  nrfUri: http://127.0.0.10:8000
  nrfCertPem: cert/nrf.pem
  supportedPlmnList:
    - mcc: 001
      mnc: 01
  supportedNssaiInPlmnList:
    - plmnId:
        mcc: 001
        mnc: 01
      supportedSnssaiList:
        - sst: 1
          sd: 000001
        - sst: 1
          sd: 000002
  nsiList:
    - snssai:
        sst: 1
        sd: 000001
      nsiInformationList:
        - nrfId: http://127.0.0.10:8000/nnrf-nfm/v1/nf-instances
          nsiId: 1
    - snssai:
        sst: 1
        sd: 000002
      nsiInformationList:
        - nrfId: http://127.0.0.10:8000/nnrf-nfm/v1/nf-instances
          nsiId: 2
  amfSetList:
    - amfSetId: 1
      amfList:
      nrfAmfSet: http://127.0.0.10:8000/nnrf-nfm/v1/nf-instances
      supportedNssaiAvailabilityData:
        - tai:
            plmnId:
              mcc: 001
              mnc: 01
            tac: 1
          supportedSnssaiList:
            - sst: 1
              sd: 000001
            - sst: 1
              sd: 000002
  taList:
    - tai:
        plmnId:
          mcc: 001
          mnc: 01
        tac: 1
      accessType: 3GPP_ACCESS
      supportedSnssaiList:
        - sst: 1
          sd: 000001
        - sst: 1
          sd: 000002

logger:
  enable: true
  level: info
  reportCaller: false
```
- `free5gc/config/smfcfg1.yaml`
```diff
--- smfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ smfcfg1.yaml        2024-03-30 11:38:45.423548870 +0900
@@ -19,60 +19,42 @@
   snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.142 # the Node ID of this SMF
+    listenAddr: 192.168.0.142 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.142 # the IP/FQDN of N4 interface on this SMF (PFCP)
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.144 # the Node ID of this UPF
+        addr: 192.168.0.144 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
                   - cidr: 10.60.0.0/16
                 staticPools:
                   - cidr: 10.60.100.0/24
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.61.0.0/16
-                staticPools:
-                  - cidr: 10.61.100.0/24
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.144
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -93,6 +75,7 @@
   urrPeriod: 10 # default usage report period in seconds
   urrThreshold: 1000 # default usage report threshold in bytes
   requestedUnit: 1000
+  ulcl: false
 logger: # log output setting
   enable: true # true or false
   level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/smfcfg2.yaml`
```diff
--- smfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ smfcfg2.yaml        2024-03-30 11:38:02.436216219 +0900
@@ -6,8 +6,8 @@
   smfName: SMF # the name of this SMF
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.2 # IP used to register to NRF
-    bindingIPv4: 127.0.0.2 # IP used to bind the service
+    registerIPv4: 127.0.0.12 # IP used to register to NRF
+    bindingIPv4: 127.0.0.12 # IP used to bind the service
     port: 8000 # Port used to bind the service
     tls: # the local path of TLS key
       key: cert/smf.key # SMF TLS Certificate
@@ -19,50 +19,32 @@
   snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000002 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.143 # the Node ID of this SMF
+    listenAddr: 192.168.0.143 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.143 # the IP/FQDN of N4 interface on this SMF (PFCP)
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.145 # the Node ID of this UPF
+        addr: 192.168.0.145 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.60.0.0/16
-                staticPools:
-                  - cidr: 10.60.100.0/24
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 000002 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
@@ -72,7 +54,7 @@
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.145
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -93,6 +75,7 @@
   urrPeriod: 10 # default usage report period in seconds
   urrThreshold: 1000 # default usage report threshold in bytes
   requestedUnit: 1000
+  ulcl: false
 logger: # log output setting
   enable: true # true or false
   level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```

<a id="changes_up1"></a>

### Changes in configuration files of free5GC 5GC U-Plane1

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-03-30 11:20:35.082415729 +0900
+++ upfcfg.yaml 2024-03-30 11:25:49.066076840 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.144   # IP addr for listening
+  nodeID: 192.168.0.144 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,7 +13,7 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.144
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -22,7 +22,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_up2"></a>

### Changes in configuration files of free5GC 5GC U-Plane2

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-03-30 11:20:35.082415729 +0900
+++ upfcfg.yaml 2024-03-30 11:35:08.624145608 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.145   # IP addr for listening
+  nodeID: 192.168.0.145 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,7 +13,7 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.145
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -22,7 +22,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN (gNodeB)

- `UERANSIM/config/free5gc-gnb.yaml`
```diff
--- free5gc-gnb.yaml.orig       2021-02-11 11:03:28.000000000 +0900
+++ free5gc-gnb.yaml    2022-08-12 18:37:46.000000000 +0900
@@ -1,23 +1,25 @@
-mcc: '208'          # Mobile Country Code value
-mnc: '93'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.1
+  - address: 192.168.0.141
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
 slices:
-  - sst: 0x1
-    sd: 0x010203
+  - sst: 1
+    sd: 0x000001
+  - sst: 1
+    sd: 0x000002
 
 # Indicates whether or not SCTP stream number errors should be ignored.
 ignoreStreamIds: true
```

<a id="changes_ue_sd1"></a>

#### Changes in configuration files of UE[SST:1, SD:0x000001] (IMSI-001010000000000)

- `UERANSIM/config/free5gc-ue-sd1.yaml`
```diff
--- free5gc-ue.yaml.orig        2024-03-02 20:20:59.000000000 +0900
+++ free5gc-ue-sd1.yaml 2024-03-30 11:54:34.811176230 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -49,18 +49,18 @@
   - type: 'IPv4'
     apn: 'internet'
     slice:
-      sst: 0x01
-      sd: 0x010203
+      sst: 1
+      sd: 0x000001
 
 # Configured NSSAI for this UE by HPLMN
 configured-nssai:
-  - sst: 0x01
-    sd: 0x010203
+  - sst: 1
+    sd: 0x000001
 
 # Default Configured NSSAI for this UE
 default-nssai:
   - sst: 1
-    sd: 1
+    sd: 0x000001
 
 # Supported integrity algorithms by this UE
 integrity:
```

<a id="changes_ue_sd2"></a>

#### Changes in configuration files of UE[SST:1, SD:0x000002] (IMSI-001010000000000)

- `UERANSIM/config/free5gc-ue-sd2.yaml`
```diff
--- free5gc-ue.yaml.orig        2024-03-02 20:20:59.000000000 +0900
+++ free5gc-ue-sd2.yaml 2024-03-30 11:55:32.709906002 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -49,18 +49,18 @@
   - type: 'IPv4'
     apn: 'internet'
     slice:
-      sst: 0x01
-      sd: 0x010203
+      sst: 1
+      sd: 0x000002
 
 # Configured NSSAI for this UE by HPLMN
 configured-nssai:
-  - sst: 0x01
-    sd: 0x010203
+  - sst: 1
+    sd: 0x000002
 
 # Default Configured NSSAI for this UE
 default-nssai:
   - sst: 1
-    sd: 1
+    sd: 0x000002
 
 # Supported integrity algorithms by this UE
 integrity:
```

<a id="network_settings"></a>

## Network settings of free5GC 5GC and UERANSIM UE / RAN

<a id="network_settings_cp"></a>

### Network settings of free5GC 5GC C-Plane

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.142/24 dev enp0s8
ip addr add 192.168.0.143/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<a id="network_settings_up1"></a>

### Network settings of free5GC 5GC U-Plane1

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="network_settings_up2"></a>

### Network settings of free5GC 5GC U-Plane2

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.61.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="build"></a>

## Build free5GC and UERANSIM

**Note. It is recommended to use go1.18.x according to the commit to free5gc/openapi on 2022.10.26.**

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. The installation guide also includes instructions on building the latest committed version.**

<a id="run"></a>

## Run free5GC 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_up"></a>

### Run free5GC 5GC U-Plane1 & U-Plane2

First, run free5GC 5GC U-Planes. Please see [here](https://github.com/free5gc/free5gc/issues/170#issuecomment-773214169) for the reason.  
**Note. It was improved on 2022.11.08, and you don't have to worry about the startup order of C-Plane and U-Plane.**

- free5GC 5GC U-Plane1
```
# cd free5gc
# bin/upf
```
- free5GC 5GC U-Plane2
```
# cd free5gc
# bin/upf
```
Then run `tcpdump` on one more terminal for each U-Plane.
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), capture size 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane2)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), capture size 262144 bytes
```

<a id="run_cp"></a>

### Run free5GC 5GC C-Plane

Next, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```bash
#!/usr/bin/env bash

PID_LIST=()

NF_LIST1="smf"
NF_LIST2="amf udr pcf udm nssf ausf chf"

export GIN_MODE=release

./bin/nrf &
PID_LIST+=($!)
sleep 1

for NF in ${NF_LIST1}; do
    ./bin/${NF} -c config/${NF}cfg1.yaml &
    PID_LIST+=($!)
    sleep 1
    ./bin/${NF} -c config/${NF}cfg2.yaml &
    PID_LIST+=($!)
    sleep 1
done

for NF in ${NF_LIST2}; do
    ./bin/${NF} &
    PID_LIST+=($!)
    sleep 1
done

function terminate()
{
    sudo kill -SIGTERM ${PID_LIST[${#PID_LIST[@]}-2]} ${PID_LIST[${#PID_LIST[@]}-1]}
    sleep 2
}

trap terminate SIGINT
wait ${PID_LIST}
```

<a id="run_ran"></a>

### Run UERANSIM (gNodeB)

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
UERANSIM v3.2.6
[2024-03-31 14:32:35.459] [sctp] [info] Trying to establish SCTP connection... (192.168.0.141:38412)
[2024-03-31 14:32:35.471] [sctp] [info] SCTP connection established (192.168.0.141:38412)
[2024-03-31 14:32:35.472] [sctp] [debug] SCTP association setup ascId[4]
[2024-03-31 14:32:35.472] [ngap] [debug] Sending NG Setup Request
[2024-03-31 14:32:35.480] [ngap] [debug] NG Setup Response received
[2024-03-31 14:32:35.480] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T14:32:35.613978916+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.131:44155
2024-03-31T14:32:35.616136448+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.131:44155
2024-03-31T14:32:35.619055524+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle NGSetupRequest
2024-03-31T14:32:35.619526576+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Send NG-Setup response
```

<a id="run_sd1"></a>

### Run UERANSIM (UE[SST:1, SD:0x000001])

Confirm that the packet goes through the DN of U-Plane1 based on SST:1 and SD:0x000001.

<a id="con_sd1"></a>

#### UE connects to U-Plane1 based on SST:1 and SD:0x000001

```
# ./nr-ue -c ../config/free5gc-ue-sd1.yaml
UERANSIM v3.2.6
[2024-03-31 14:33:24.236] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 14:33:24.238] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 14:33:25.332] [nas] [info] Selected plmn[001/01]
[2024-03-31 14:33:25.333] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-31 14:33:25.333] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 14:33:25.334] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 14:33:25.336] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 14:33:25.337] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 14:33:25.338] [nas] [debug] Sending Initial Registration
[2024-03-31 14:33:25.339] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 14:33:25.339] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 14:33:25.341] [rrc] [info] RRC connection established
[2024-03-31 14:33:25.342] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 14:33:25.342] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 14:33:25.462] [nas] [debug] Authentication Request received
[2024-03-31 14:33:25.463] [nas] [debug] Received SQN [000000000029]
[2024-03-31 14:33:25.463] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 14:33:25.489] [nas] [debug] Security Mode Command received
[2024-03-31 14:33:25.489] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 14:33:25.650] [nas] [debug] Registration accept received
[2024-03-31 14:33:25.650] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 14:33:25.651] [nas] [debug] Sending Registration Complete
[2024-03-31 14:33:25.651] [nas] [info] Initial Registration is successful
[2024-03-31 14:33:25.651] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 14:33:25.651] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 14:33:25.864] [nas] [debug] Configuration Update Command received
[2024-03-31 14:33:26.064] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 14:33:26.068] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 14:33:26.091] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T14:33:25.088115022+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle InitialUEMessage
2024-03-31T14:33:25.088965374+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-03-31T14:33:25.089762670+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T14:33:25.094747889+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-03-31T14:33:25.095976700+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T14:33:25.096824835+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T14:33:25.097668663+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T14:33:25.098174885+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T14:33:25.098914874+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T14:33:25.099724798+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T14:33:25.102595164+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.110078598+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.121425079+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.125415299+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.128478672+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T14:33:25.130440903+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.138175600+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.148246624+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.152202800+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T14:33:25.152578557+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T14:33:25.153869355+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.156453585+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.161618187+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.163494550+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.165541002+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T14:33:25.166460301+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.167890339+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.173761324+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.176500798+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T14:33:25.177467387+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.179372150+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.184018940+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.184508119+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T14:33:25.184652722+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T14:33:25.185041153+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-03-31T14:33:25.186659449+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.188257329+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.191550533+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.192906329+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.193824484+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-03-31T14:33:25.195836136+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T14:33:25.197821789+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T14:33:25.198916015+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T14:33:25.200676192+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T14:33:25.202377435+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T14:33:25.202750303+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T14:33:25.203225544+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T14:33:25.203292442+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T14:33:25.203326835+09:00 [INFO][AUSF][5gAka] XresStar = 3537383366386463356535653734663130643766326331616662343638343665
2024-03-31T14:33:25.203434249+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T14:33:25.203893506+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T14:33:25.204022069+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:33:25.204361698+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T14:33:25.206143550+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:33:25.206252973+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.206521998+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T14:33:25.206613993+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T14:33:25.206774119+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T14:33:25.207665316+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.209154180+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.212251428+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.213704773+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T14:33:25.213812316+09:00 [INFO][AUSF][5gAka] res*: 3537383366386463356535653734663130643766326331616662343638343665
Xres*: 3537383366386463356535653734663130643766326331616662343638343665
2024-03-31T14:33:25.214357303+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T14:33:25.215023405+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.215918843+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.219530249+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.220973539+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T14:33:25.221750909+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.223053987+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.226037716+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.227701119+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T14:33:25.228722108+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T14:33:25.229005047+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T14:33:25.229478185+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T14:33:25.229806335+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T14:33:25.229895623+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T14:33:25.229949006+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:33:25.230534523+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T14:33:25.233129711+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:33:25.233321121+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.233531032+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T14:33:25.233622490+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T14:33:25.233780025+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T14:33:25.234353619+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T14:33:25.234446310+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T14:33:25.235277766+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.236887896+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.239686829+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.240727493+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.241995027+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T14:33:25.242500658+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.243865957+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.247334095+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.249406350+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T14:33:25.251867730+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.253301358+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.257072067+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.259986642+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T14:33:25.260702610+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T14:33:25.261151554+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:33:25.261558079+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 0 0 1]}
2024-03-31T14:33:25.261751467+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2024-03-31T14:33:25.262739548+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.264084489+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.266566225+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.267857087+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.268890237+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T14:33:25.269625895+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.270795006+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.274124479+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.275809066+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T14:33:25.275916207+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T14:33:25.276715742+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.278179598+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.281402964+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.282992718+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T14:33:25.284170409+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T14:33:25.284567517+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T14:33:25.286015448+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.287369108+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.290979011+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.292421833+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T14:33:25.292992699+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.294091841+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.297291656+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.298559574+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T14:33:25.299198175+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:33:25.299615782+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:33:25.300877497+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.302442715+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.305708904+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.307086627+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T14:33:25.307726504+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.308881247+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.311900625+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.313234782+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T14:33:25.313849970+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T14:33:25.314195696+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:33:25.315606564+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.316928006+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.320455222+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.321816585+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T14:33:25.322329889+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.323477313+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.326520499+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.328005304+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T14:33:25.328529424+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T14:33:25.329068050+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T14:33:25.330034308+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.332585109+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.337116647+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.338858598+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T14:33:25.339651913+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.340796933+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.343900715+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.345238141+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T14:33:25.345544261+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T14:33:25.345867922+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T14:33:25.347066031+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.348362187+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.350978153+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.351986988+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.353180835+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T14:33:25.353865747+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.355046918+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.358502065+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.360901509+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T14:33:25.361590868+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.362888948+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.365515093+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.366435407+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.367301328+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T14:33:25.367859095+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.369064477+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.372216214+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.373652866+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T14:33:25.374174168+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T14:33:25.375221610+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.376667692+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.379198327+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.380154380+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.381499728+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T14:33:25.382031733+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.383291697+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.386964870+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.388478108+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-03-31T14:33:25.388798918+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2024-03-31T14:33:25.388990867+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-03-31T14:33:25.389523418+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T14:33:25.390152203+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T14:33:25.390358693+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send Initial Context Setup Request
2024-03-31T14:33:25.391806802+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T14:33:25.392696523+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle InitialContextSetupResponse
2024-03-31T14:33:25.392857592+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.597250377+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:33:25.598110235+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.598933972+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T14:33:25.599561239+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T14:33:25.599851641+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T14:33:25.600771321+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T14:33:25.601511037+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:33:25.604972629+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T14:33:25.608479756+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:33:25.609107330+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.609823045+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T14:33:25.610377994+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T14:33:25.611204168+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T14:33:25.611514400+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2024-03-31T14:33:25.614810980+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.621361214+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.632020267+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.635493839+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.638764720+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T14:33:25.640276581+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.643602128+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.652056033+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.655584021+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T14:33:25.656324736+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=999e958e-0baf-4d09-9500-4e250eb56d4f&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T14:33:25.659771127+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.665049922+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.671454039+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.673083676+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.675333367+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:33:25.676116547+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.678155258+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.682901528+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.685633648+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T14:33:25.686557131+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T14:33:25.686811101+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T14:33:25.686942478+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T14:33:25.687866752+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.689444842+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.693068968+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.694646492+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.696319619+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T14:33:25.696812628+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T14:33:25.697396304+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.698502967+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.702648033+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.704183694+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T14:33:25.705174983+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.706559493+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.710080861+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.710480738+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2024-03-31T14:33:25.711948888+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T14:33:25.712909765+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T14:33:25.713345311+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T14:33:25.713917881+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T14:33:25+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc000308600 0xc000308620]
2024-03-31T14:33:25.713987092+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T14:33:25.714008986+09:00 [INFO][SMF][GSM] &{[0xc000308600 0xc000308620]}
2024-03-31T14:33:25.714023376+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T14:33:25.714451415+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.715831447+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.718508801+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.719692101+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.720898998+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=999e958e-0baf-4d09-9500-4e250eb56d4f&target-nf-type=AMF |
2024-03-31T14:33:25.721327340+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T14:33:25.721660858+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-03-31T14:33:25.721837919+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T14:33:25.721987480+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2024-03-31T14:33:25.722923154+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.723875111+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.726631322+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.727532962+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.728696165+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T14:33:25.729295007+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.730286497+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.733544209+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.734750615+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T14:33:25.735352272+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.736552805+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.739585973+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.740897293+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T14:33:25.742008245+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T14:33:25.745198665+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.746527925+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.749667646+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.750893786+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T14:33:25.751575562+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&supis=imsi-001010000000000 |
2024-03-31T14:33:25.751883453+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T14:33:25.753195898+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.755529353+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.759986663+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.761488180+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T14:33:25.761677454+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T14:33:25.762118769+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.763319973+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.765929441+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.767510208+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.768068199+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T14:33:25.768818472+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T14:33:25.770529023+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T14:33:25.773179840+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.774305114+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.776880108+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.778133685+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:33:25.778954487+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T14:33:25.779276471+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T14:33:25.779676308+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.780632142+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.783729396+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.787733531+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T14:33:25.787941166+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T14:33:25.788211987+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T14:33:25.788385230+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T14:33:25.788755924+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T14:33:25.789234324+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T14:33:25.789807350+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T14:33:25.789990302+09:00 [ERRO][SMF][CTX] No default data path
2024-03-31T14:33:25.790239955+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has no default path
2024-03-31T14:33:25.790660289+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T14:33:25.790841785+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T14:33:25.791752023+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T14:33:25.794799304+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T14:33:25.795944729+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.797448528+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.800834969+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.802817587+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T14:33:25.803057940+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send PDU Session Resource Setup Request
2024-03-31T14:33:25.804102182+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T14:33:25.806341109+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle PDUSessionResourceSetupResponse
2024-03-31T14:33:25.806490228+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T14:33:25.807245573+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:33:25.808736203+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:33:25.812146563+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:33:25.813889764+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T14:33:25.815636774+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T14:33:25.815869086+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:29098c39-98d2-4ebe-881d-228e331991dd/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2024-03-31T14:33:25.935801360+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionEstablishmentRequest
2024-03-31T14:33:25.935835221+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T14:33:25.936994929+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T14:33:25.937471064+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T14:33:25.958965488+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
6: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::b1bb:2d80:703c:a0fe/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_sd1"></a>

#### Ping google.com going through DN=10.60.0.0/16 on U-Plane1

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.175.110) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.175.110: icmp_seq=1 ttl=61 time=26.6 ms
64 bytes from 172.217.175.110: icmp_seq=2 ttl=61 time=24.1 ms
64 bytes from 172.217.175.110: icmp_seq=3 ttl=61 time=19.3 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
14:37:07.003862 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 4, seq 1, length 64
14:37:07.027647 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 4, seq 1, length 64
14:37:08.004848 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 4, seq 2, length 64
14:37:08.026409 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 4, seq 2, length 64
14:37:09.006869 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 4, seq 3, length 64
14:37:09.023399 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 4, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2.**

<a id="run_sd2"></a>

### Run UERANSIM (UE[SST:1, SD:0x000002])

Then the UE disconnects from gNodeB and connects to gNodeB using the configuration file for SST:1 and SD:0x000002.
Confirm that the packet goes through the DN of U-Plane2 based on SST:1 and SD:0x000002.

<a id="con_sd2"></a>

#### UE connects to U-Plane2 based on SST:1 and SD:0x000002

```
# ./nr-ue -c ../config/free5gc-ue-sd2.yaml 
UERANSIM v3.2.6
[2024-03-31 14:38:24.562] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 14:38:24.565] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 14:38:25.856] [nas] [info] Selected plmn[001/01]
[2024-03-31 14:38:25.857] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-31 14:38:25.858] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 14:38:25.859] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 14:38:25.859] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 14:38:25.860] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 14:38:25.860] [nas] [debug] Sending Initial Registration
[2024-03-31 14:38:25.861] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 14:38:25.862] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 14:38:25.863] [rrc] [info] RRC connection established
[2024-03-31 14:38:25.864] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 14:38:25.865] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 14:38:25.984] [nas] [debug] Authentication Request received
[2024-03-31 14:38:25.985] [nas] [debug] Received SQN [00000000002A]
[2024-03-31 14:38:25.985] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 14:38:26.021] [nas] [debug] Security Mode Command received
[2024-03-31 14:38:26.021] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 14:38:26.051] [nas] [debug] Registration accept received
[2024-03-31 14:38:26.051] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 14:38:26.051] [nas] [debug] Sending Registration Complete
[2024-03-31 14:38:26.051] [nas] [info] Initial Registration is successful
[2024-03-31 14:38:26.051] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 14:38:26.052] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 14:38:26.295] [nas] [debug] Configuration Update Command received
[2024-03-31 14:38:26.433] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 14:38:26.437] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 14:38:26.460] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T14:38:25.601264355+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle InitialUEMessage
2024-03-31T14:38:25.602299658+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] New RanUe [RanUeNgapID:2][AmfUeNgapID:2]
2024-03-31T14:38:25.603066736+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T14:38:25.603798296+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] find AmfUe ["SUCI":"suci-0-001-01-0000-0-0-0000000000"]
2024-03-31T14:38:25.604445396+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Implicit Deregistration - RanUeNgapID[1]
2024-03-31T14:38:25.604955055+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Send UE Context Release Command
2024-03-31T14:38:25.607106637+09:00 [WARN][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] AmfUe is nil
2024-03-31T14:38:25.610638214+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T14:38:25.611556529+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Request
2024-03-31T14:38:25.612398322+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] RegistrationType: Initial Registration
2024-03-31T14:38:25.613003580+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T14:38:25.613909917+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Registered] to [Authentication]
2024-03-31T14:38:25.614532055+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Authentication procedure
2024-03-31T14:38:25.617655208+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.624152664+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.634669912+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.638916599+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:25.642619708+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T14:38:25.644816444+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.648845054+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.657328160+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.661814783+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T14:38:25.662774861+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T14:38:25.664445845+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.666828668+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.674039253+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.676485289+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:25.679243463+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T14:38:25.680548856+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.682764708+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.691289354+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.694854398+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T14:38:25.696301997+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.699272334+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.705635352+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.706216536+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T14:38:25.706499880+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T14:38:25.706781423+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2024-03-31T14:38:25.709266530+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T14:38:25.710312779+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T14:38:25.710941981+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T14:38:25.712430185+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T14:38:25.715441090+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T14:38:25.715883087+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T14:38:25.716561687+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T14:38:25.716600134+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T14:38:25.716621032+09:00 [INFO][AUSF][5gAka] XresStar = 6136656431383939636136313436646565613035363239643432643662626663
2024-03-31T14:38:25.716686931+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T14:38:25.717028878+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Send Authentication Request
2024-03-31T14:38:25.717116287+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:38:25.717773993+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T14:38:25.719096890+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UEContextReleaseComplete
2024-03-31T14:38:25.719364881+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44155] Handle UEContextReleaseComplete (RAN UE NGAP ID: 1)
2024-03-31T14:38:25.719700361+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Release UE Context : RanUe[AmfUeNgapId: 1]
2024-03-31T14:38:25.721557877+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:38:25.724148215+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 2)
2024-03-31T14:38:25.724470812+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T14:38:25.725020334+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle Authentication Response
2024-03-31T14:38:25.725255228+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T14:38:25.726251421+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.728386156+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.732597791+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.734293029+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T14:38:25.734707484+09:00 [INFO][AUSF][5gAka] res*: 6136656431383939636136313436646565613035363239643432643662626663
Xres*: 6136656431383939636136313436646565613035363239643432643662626663
2024-03-31T14:38:25.735232060+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T14:38:25.735937236+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.737151113+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.741255018+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.743481795+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T14:38:25.744189487+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.745671658+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.749297019+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.750998840+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T14:38:25.752082141+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T14:38:25.752414362+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T14:38:25.752796599+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T14:38:25.753078105+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T14:38:25.753480305+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T14:38:25.753564444+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:38:25.754114632+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T14:38:25.755885462+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:38:25.756001396+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 2)
2024-03-31T14:38:25.756292980+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T14:38:25.756384348+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T14:38:25.756656762+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T14:38:25.756763430+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T14:38:25.756950602+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T14:38:25.757177979+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 0 0 2]}
2024-03-31T14:38:25.757356856+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000002}, HomeSnssai: <nil>
2024-03-31T14:38:25.758011381+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.760475743+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.763239962+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.765748093+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:25.767092046+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T14:38:25.767621959+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.768805685+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.772532197+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.774061506+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T14:38:25.774669704+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.775979669+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.779343239+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.780544856+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T14:38:25.781147061+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T14:38:25.781591434+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T14:38:25.782004971+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T14:38:25.782150059+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Send Initial Context Setup Request
2024-03-31T14:38:25.783806992+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T14:38:25.784814518+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle InitialContextSetupResponse
2024-03-31T14:38:25.784981308+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle InitialContextSetupResponse (RAN UE NGAP ID: 2)
2024-03-31T14:38:25.988843752+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:38:25.989132622+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 2)
2024-03-31T14:38:25.989361174+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T14:38:25.989468488+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T14:38:25.989725414+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T14:38:25.990484830+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.992116493+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:25.995338504+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:25.997022749+09:00 [INFO][SMF][PduSess] Receive Release SM Context Request
2024-03-31T14:38:25.997312018+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextRelease
2024-03-31T14:38:25.997979919+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:25.999137817+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.004189818+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.005147167+09:00 [INFO][PCF][SMpolicy] Handle DeleteSmPolicyContext
2024-03-31T14:38:26.006114948+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.007433602+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.010786647+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.012127787+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifySubscriptionIdDelete
2024-03-31T14:38:26.012366217+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | DELETE  | /nudr-dr/v1/application-data/influenceData/subs-to-notify/a4659a47 |
2024-03-31T14:38:26.012634023+09:00 [INFO][PCF][GIN] | 204 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies/imsi-001010000000000-1/delete |
2024-03-31T14:38:26.013024194+09:00 [WARN][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Unexpected state, expect: [InActive], actual:[Active]
2024-03-31T14:38:26.013252204+09:00 [INFO][SMF][PduSess] Sending PFCP Session Deletion Request
2024-03-31T14:38:26.018428675+09:00 [INFO][SMF][PduSess] Received PFCP Session Deletion Accepted Response
2024-03-31T14:38:26.018575763+09:00 [INFO][SMF][Charging] build MultiUnitUsageFromUsageReport
2024-03-31T14:38:26.018864259+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T14:38:26.019522196+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.020856501+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.023972978+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.025484821+09:00 [INFO][CHF][ChargingPost] HandleChargingdateRelease
2024-03-31T14:38:26.025787477+09:00 [INFO][CHF][ChargingPost] Close CDR
2024-03-31T14:38:26.026103102+09:00 [INFO][CHF][GIN] | 204 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata/imsi-001010000000000SMF0/release |
2024-03-31T14:38:26.026470093+09:00 [INFO][SMF][Charging] Send Charging Data Request[Termination] successfully
2024-03-31T14:38:26.026581085+09:00 [INFO][SMF][GIN] | 204 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:29098c39-98d2-4ebe-881d-228e331991dd/release |
2024-03-31T14:38:26.026913860+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] Release IP[10.60.0.1]
2024-03-31T14:38:26.027162427+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] smContext[urn:uuid:29098c39-98d2-4ebe-881d-228e331991dd] is deleted from pool
2024-03-31T14:38:26.027026949+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T14:38:26.027458963+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Send Downlink Nas Transport
2024-03-31T14:38:26.028400885+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T14:38:26.031049660+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport
2024-03-31T14:38:26.031336255+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle UplinkNASTransport (RAN UE NGAP ID: 2)
2024-03-31T14:38:26.031594649+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T14:38:26.031687395+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T14:38:26.031879980+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T14:38:26.032038419+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000002}, dnn: internet]
2024-03-31T14:38:26.032991186+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.034579246+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.038202078+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.039718725+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T14:38:26.039925677+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=999e958e-0baf-4d09-9500-4e250eb56d4f&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T14:38:26.041082419+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.042617238+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.045217109+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.046414006+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.047611719+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T14:38:26.048191233+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.050121480+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.054344387+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.057066836+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T14:38:26.057842928+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T14:38:26.058061413+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T14:38:26.058234394+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T14:38:26.058812766+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.060092043+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.062777516+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.063967442+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.065094686+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T14:38:26.065756930+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T14:38:26.066156463+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.067162714+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.070796388+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.072142160+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T14:38:26.072817226+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.073950881+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.077106102+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.077605416+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000002"}]
2024-03-31T14:38:26.078772816+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T14:38:26.079446969+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2024-03-31T14:38:26.080115865+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2024-03-31T14:38:26.080650274+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T14:38:26+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0001d87a0 0xc0001d87c0]
2024-03-31T14:38:26.080693263+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T14:38:26.080713538+09:00 [INFO][SMF][GSM] &{[0xc0001d87a0 0xc0001d87c0]}
2024-03-31T14:38:26.080726723+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T14:38:26.081268839+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.082649193+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.085238057+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.086243649+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.087382455+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=999e958e-0baf-4d09-9500-4e250eb56d4f&target-nf-type=AMF |
2024-03-31T14:38:26.087866256+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T14:38:26.088175033+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2024-03-31T14:38:26.088357987+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T14:38:26.088536186+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.61.0.1]
2024-03-31T14:38:26.088897317+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.089752357+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.092265042+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.093763972+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.094955107+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T14:38:26.095627132+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.096545037+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.100069522+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.101125914+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T14:38:26.101813454+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.103005093+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.106111663+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.107365876+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T14:38:26.108411151+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2024-03-31T14:38:26.110525784+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.111796167+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.114711864+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.116002681+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T14:38:26.116430190+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D&supis=imsi-001010000000000 |
2024-03-31T14:38:26.116869799+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T14:38:26.117605546+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.118931035+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.122323803+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.123735922+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T14:38:26.123907749+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T14:38:26.124852408+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.126129547+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.128770270+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.129762069+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.130389899+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T14:38:26.130692852+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T14:38:26.131630176+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T14:38:26.132226340+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.134092719+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.136790608+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.139431751+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T14:38:26.140394758+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T14:38:26.140784896+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T14:38:26.141151562+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.142151022+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.145059920+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.147414733+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T14:38:26.147724907+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T14:38:26.147967149+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T14:38:26.148225709+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T14:38:26.148367675+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T14:38:26.148572380+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T14:38:26.149209571+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T14:38:26.149404274+09:00 [ERRO][SMF][CTX] No default data path
2024-03-31T14:38:26.149551781+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has no default path
2024-03-31T14:38:26.149860527+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T14:38:26.150226731+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T14:38:26.150913522+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:2,AU:2(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T14:38:26.153538025+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T14:38:26.155380213+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.158400033+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.162043005+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.163642900+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T14:38:26.163980447+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Send PDU Session Resource Setup Request
2024-03-31T14:38:26.164894632+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T14:38:26.167112682+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44155] Handle PDUSessionResourceSetupResponse
2024-03-31T14:38:26.167300188+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:2,AU:2(3GPP)][ran_addr:192.168.0.131:44155] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 2)
2024-03-31T14:38:26.167875268+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T14:38:26.169422873+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T14:38:26.172858321+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T14:38:26.174362264+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T14:38:26.176902169+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T14:38:26.177166064+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:71c4ba19-0c13-4c31-a1c2-1ae76b12700a/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2024-03-31T14:38:26.065181761+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionEstablishmentRequest
2024-03-31T14:38:26.065242857+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805][CPNodeID:192.168.0.143][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T14:38:26.066256880+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T14:38:26.066623400+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T14:38:26.089661944+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::63d9:cd3b:7d8d:d7c3/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_sd2"></a>

#### Ping google.com going through DN=10.61.0.0/16 on U-Plane2

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.161.78) from 10.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.161.78: icmp_seq=1 ttl=61 time=21.7 ms
64 bytes from 172.217.161.78: icmp_seq=2 ttl=61 time=19.6 ms
64 bytes from 172.217.161.78: icmp_seq=3 ttl=61 time=19.6 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
14:43:04.125640 IP 10.61.0.1 > 172.217.161.78: ICMP echo request, id 5, seq 1, length 64
14:43:04.145347 IP 172.217.161.78 > 10.61.0.1: ICMP echo reply, id 5, seq 1, length 64
14:43:05.126865 IP 10.61.0.1 > 172.217.161.78: ICMP echo request, id 5, seq 2, length 64
14:43:05.143745 IP 172.217.161.78 > 10.61.0.1: ICMP echo reply, id 5, seq 2, length 64
14:43:06.127314 IP 10.61.0.1 > 172.217.161.78: ICMP echo request, id 5, seq 3, length 64
14:43:06.144716 IP 172.217.161.78 > 10.61.0.1: ICMP echo reply, id 5, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.31] Updated to free5GC v3.4.1 (2024.03.28).
- [2023.03.18] Updated to free5GC v3.2.1 (2023.02.12) and UERANSIM v3.2.6 (2023.03.17).
- [2022.08.12] Initial release.

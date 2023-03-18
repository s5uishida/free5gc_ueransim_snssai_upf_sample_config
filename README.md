# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses free5GC and UERANSIM to select the UPF based on S-NSSAI.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/free5gc_ueransim_sample_config)
2. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/free5gc_ueransim_nearby_upf_sample_config)
3. Select UPF based on S-NSSAI (this article)
4. [ULCL(Uplink Classifier)](https://github.com/s5uishida/free5gc_ueransim_ulcl_sample_config)
5. [ULCL with one I-UPF and two PSA-UPFs](https://github.com/s5uishida/free5gc_ueransim_ulcl_2_sample_config)

---

<h2 id="misc">Miscellaneous Notes</h2>

- [Install MongoDB 6.0 and free5GC WebUI](https://github.com/s5uishida/free5gc_install_mongodb6_webui)

---

<h2 id="toc">Table of Contents</h2>

- [Overview of free5GC 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of free5GC 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of free5GC 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of free5GC 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB)](#changes_ran)
    - [Changes in configuration files of UE set to SST:1 and SD:0x000001 (IMSI-001010000000000)](#changes_ue_sd1)
    - [Changes in configuration files of UE set to SST:1 and SD:0x000002 (IMSI-001010000000000)](#changes_ue_sd2)
- [Network settings of free5GC 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of free5GC 5GC C-Plane](#network_settings_cp)
  - [Network settings of free5GC 5GC U-Plane1](#network_settings_up1)
  - [Network settings of free5GC 5GC U-Plane2](#network_settings_up2)
- [Build free5GC and UERANSIM](#build)
- [Run free5GC 5GC and UERANSIM UE / RAN](#run)
  - [Run free5GC 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run free5GC 5GC C-Plane](#run_cp)
  - [Run UERANSIM (gNodeB)](#run_ran)
  - [Run UERANSIM (UE[SST:1, SD:0x000001] connects to U-Plane1)](#run_sd1)
  - [Run UERANSIM (UE[SST:1, SD:0x000002] connects to U-Plane2)](#run_sd2)
  - [Ping google.com going through DN=10.60.0.0/16 on U-Plane1 via uesimtun0](#ping_sd1)
  - [Ping google.com going through DN=10.61.0.0/16 on U-Plane2 via uesimtun1](#ping_sd2)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of free5GC 5GC Simulation Mobile Network</h2>

The following minimum configuration was set as a condition.
- The UE selects a pair of SMF and UPF based on S-NSSAI.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v3.2.1 (2023.02.12) - https://github.com/free5gc/free5gc
- UE / RAN - UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM

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
| 10.61.0.0/16 | SST:1 <br> SD:0x000002 | upfgtp | internet | uesimtun1 | U-Plane2 |

<h2 id="changes">Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.2.1 (2023.02.12) - https://github.com/free5gc/free5gc/wiki/Installation
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of free5GC 5GC C-Plane</h3>

- `free5gc/config/amfcfg.yaml`
```diff
--- amfcfg.yaml.orig    2022-04-01 20:25:54.000000000 +0900
+++ amfcfg.yaml 2022-08-12 18:07:50.000000000 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.141
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
     registerIPv4: 127.0.0.18 # IP used to register to NRF
@@ -23,23 +23,23 @@
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
       tac: 1 # Tracking Area Code (uinteger, range: 0~16777215)
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
- `free5gc/config/smfcfg1.yaml`
```diff
--- smfcfg.yaml.orig    2023-03-18 02:14:22.134727451 +0900
+++ smfcfg1.yaml        2023-03-18 01:06:11.448054534 +0900
@@ -19,55 +19,42 @@
   snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this SMF
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.142
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.144 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
                   - cidr: 10.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.61.0.0/16
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.144
             networkInstance: internet # Data Network Name (DNN)
     links: # the topology graph of userplane, A and B represent the two nodes of each link
       - A: gNB1
         B: UPF
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
+  ulcl: false
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/smfcfg2.yaml`
```diff
--- smfcfg.yaml.orig    2023-03-18 02:14:22.134727451 +0900
+++ smfcfg2.yaml        2023-03-18 01:06:21.617046237 +0900
@@ -6,8 +6,8 @@
   smfName: SMF # the name of this SMF
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.2 # IP used to register to NRF
-    bindingIPv4: 127.0.0.2  # IP used to bind the service
+    registerIPv4: 127.0.0.12 # IP used to register to NRF
+    bindingIPv4: 127.0.0.12  # IP used to bind the service
     port: 8000 # Port used to bind the service
     tls: # the local path of TLS key
       key: config/TLS/smf.key # SMF TLS Certificate
@@ -19,42 +19,28 @@
   snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this SMF
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000002 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.143
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.145 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 000002 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
@@ -62,12 +48,13 @@
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.145
             networkInstance: internet # Data Network Name (DNN)
     links: # the topology graph of userplane, A and B represent the two nodes of each link
       - A: gNB1
         B: UPF
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
+  ulcl: false
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2022-04-01 20:25:54.000000000 +0900
+++ ausfcfg.yaml        2022-08-12 18:08:52.000000000 +0900
@@ -15,10 +15,8 @@
     - nausf-auth # Nausf_UEAuthentication service
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
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
--- nrfcfg.yaml.orig    2023-03-18 00:54:05.484701280 +0900
+++ nrfcfg.yaml 2023-03-18 01:49:29.353185298 +0900
@@ -14,8 +14,8 @@
     # rootcert: config/cert/root.pem # Root Certificate
     # oauth: false # OAuth2
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
  version: 1.0.1
  description: NSSF initial local configuration

configuration:
  nssfName: NSSF
  sbi:
    scheme: http
    registerIPv4: 127.0.0.31
    bindingIPv4: 127.0.0.31
    port: 8000
    tls:
      pem: config/TLS/nssf.pem
      key: config/TLS/nssf.key
  serviceNameList:
    - nnssf-nsselection
    - nnssf-nssaiavailability
  nrfUri: http://127.0.0.10:8000
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
  NSSF:
    debugLevel: info
    ReportCaller: false
```

<h3 id="changes_up1">Changes in configuration files of free5GC 5GC U-Plane1</h3>

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2022-08-11 14:42:32.545805826 +0900
+++ upfcfg.yaml 2022-08-11 16:52:48.563998874 +0900
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
@@ -21,7 +21,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<h3 id="changes_up2">Changes in configuration files of free5GC 5GC U-Plane2</h3>

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2022-08-11 14:44:10.418739050 +0900
+++ upfcfg.yaml 2022-08-11 16:56:14.233871256 +0900
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
@@ -21,7 +21,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN (gNodeB)</h4>

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

<h4 id="changes_ue_sd1">Changes in configuration files of UE set to SST:1 and SD:0x000001 (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-sd1.yaml`
```diff
--- free5gc-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ free5gc-ue-sd1.yaml 2023-03-18 02:47:48.152581680 +0900
@@ -1,11 +1,11 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000003'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # Routing Indicator
-routingIndicator: '0000'
+routingIndicator: '0001'
 
 # Permanent subscription key
 key: '8baf473f2f8fd09487cccbd7097c6862'
@@ -22,7 +22,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -43,18 +43,18 @@
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

<h4 id="changes_ue_sd2">Changes in configuration files of UE set to SST:1 and SD:0x000002 (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-sd2.yaml`
```diff
--- free5gc-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ free5gc-ue-sd2.yaml 2023-03-18 02:48:04.841578308 +0900
@@ -1,11 +1,11 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000003'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # Routing Indicator
-routingIndicator: '0000'
+routingIndicator: '0002'
 
 # Permanent subscription key
 key: '8baf473f2f8fd09487cccbd7097c6862'
@@ -22,7 +22,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -43,18 +43,18 @@
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

<h2 id="network_settings">Network settings of free5GC 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_cp">Network settings of free5GC 5GC C-Plane</h3>

Add IP addresses for SMF1 and SMF2.
```
ip addr add 192.168.0.142/24 dev enp0s8
ip addr add 192.168.0.143/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<h3 id="network_settings_up1">Network settings of free5GC 5GC U-Plane1</h3>

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

<h3 id="network_settings_up2">Network settings of free5GC 5GC U-Plane2</h3>

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

<h2 id="build">Build free5GC and UERANSIM</h2>

**Note. It is recommended to use go1.18.x according to the commit to free5gc/openapi on 2022.10.26.**

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.2.1 (2023.02.12) - https://github.com/free5gc/free5gc/wiki/Installation
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. If you want to use the latest committed version, please run the following script to checkout all NFs and Web Console to the latest `main` branch before building.**
```bash
#!/usr/bin/env bash

NF_LIST="nrf amf smf udr pcf udm nssf ausf upf n3iwf"

for NF in ${NF_LIST}; do
    cd NFs/${NF}
    git checkout main
    cd ../..
done

cd webconsole
git checkout main

cd ..
git checkout main
```

<h2 id="run">Run free5GC 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_up">Run free5GC 5GC U-Plane1 & U-Plane2</h3>

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

<h3 id="run_cp">Run free5GC 5GC C-Plane</h3>

Next, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```bash
#!/usr/bin/env bash

PID_LIST=()

NF_LIST1="smf"
NF_LIST2="amf udr pcf udm nssf ausf"

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

<h3 id="run_ran">Run UERANSIM (gNodeB)</h3>

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2023-03-18 03:15:30.771] [sctp] [info] Trying to establish SCTP connection... (192.168.0.141:38412)
[2023-03-18 03:15:30.774] [sctp] [info] SCTP connection established (192.168.0.141:38412)
[2023-03-18 03:15:30.774] [sctp] [debug] SCTP association setup ascId[7]
[2023-03-18 03:15:30.774] [ngap] [debug] Sending NG Setup Request
[2023-03-18 03:15:30.777] [ngap] [debug] NG Setup Response received
[2023-03-18 03:15:30.777] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T03:15:30.773072751+09:00 [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.131:40266
2023-03-18T03:15:30.773783929+09:00 [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.131:40266
2023-03-18T03:15:30.774414669+09:00 [INFO][AMF][NGAP][192.168.0.131:40266] Handle NG Setup request
2023-03-18T03:15:30.774461012+09:00 [INFO][AMF][NGAP][192.168.0.131:40266] Send NG-Setup response
```

<h3 id="run_sd1">Run UERANSIM (UE[SST:1, SD:0x000001] connects to U-Plane1)</h3>

UE connects to U-Plane1 based on SST:1 and SD:0x000001.

```
# ./nr-ue -c ../config/free5gc-ue-sd1.yaml 
UERANSIM v3.2.6
[2023-03-18 03:18:05.376] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 03:18:05.377] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 03:18:05.377] [nas] [info] Selected plmn[001/01]
[2023-03-18 03:18:05.377] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 03:18:05.377] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 03:18:05.377] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 03:18:05.377] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 03:18:05.378] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 03:18:05.378] [nas] [debug] Sending Initial Registration
[2023-03-18 03:18:05.378] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 03:18:05.378] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 03:18:05.379] [rrc] [info] RRC connection established
[2023-03-18 03:18:05.380] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 03:18:05.380] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 03:18:05.405] [nas] [debug] Authentication Request received
[2023-03-18 03:18:05.413] [nas] [debug] Security Mode Command received
[2023-03-18 03:18:05.413] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 03:18:05.450] [nas] [debug] Registration accept received
[2023-03-18 03:18:05.451] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 03:18:05.451] [nas] [debug] Sending Registration Complete
[2023-03-18 03:18:05.451] [nas] [info] Initial Registration is successful
[2023-03-18 03:18:05.451] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 03:18:05.451] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 03:18:05.705] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 03:18:05.710] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 03:18:05.735] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T03:18:05.376487306+09:00 [INFO][AMF][NGAP][192.168.0.131:40266] Handle Initial UE Message
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2023-03-18T03:18:05.377104892+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2023-03-18T03:18:05.377145062+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2023-03-18T03:18:05.378257772+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.379299302+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2023-03-18T03:18:05.380847961+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2023-03-18T03:18:05.381095687+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2023-03-18T03:18:05.382153174+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.383850851+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2023-03-18T03:18:05.385344438+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2023-03-18T03:18:05.38584428+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0001 0 0 0000000000]
2023-03-18T03:18:05.386124513+09:00 [INFO][UDM][Suci] scheme 0
2023-03-18T03:18:05.386335013+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2023-03-18T03:18:05.387340701+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.388198325+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2023-03-18T03:18:05.389271169+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2023-03-18T03:18:05.390895213+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T03:18:05.39275831+09:00 [INFO][UDM][UEAU] Nil Op
2023-03-18T03:18:05.395250467+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2023-03-18T03:18:05.397216936+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T03:18:05.397675239+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0001-0-0-0000000000/security-information/generate-auth-data |
2023-03-18T03:18:05.398150945+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0001-0-0-0000000000, imsi-001010000000000) to map.
2023-03-18T03:18:05.39848704+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2023-03-18T03:18:05.398815278+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 3732643133643430643938613462316435313736363631366366616461646464
2023-03-18T03:18:05.399084482+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2023-03-18T03:18:05.399656491+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2023-03-18T03:18:05.399810512+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2023-03-18T03:18:05.401828585+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2023-03-18T03:18:05.402315353+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2023-03-18T03:18:05.403213848+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2023-03-18T03:18:05.403281752+09:00 [INFO][AUSF][5gAkaAuth] res*: 3732643133643430643938613462316435313736363631366366616461646464
Xres*: 3732643133643430643938613462316435313736363631366366616461646464
2023-03-18T03:18:05.403367514+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2023-03-18T03:18:05.403994657+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2023-03-18T03:18:05.404763027+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2023-03-18T03:18:05.406321641+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2023-03-18T03:18:05.406651108+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2023-03-18T03:18:05.407176064+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0001-0-0-0000000000/5g-aka-confirmation |
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2023-03-18T03:18:05.40778455+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2023-03-18T03:18:05.407854322+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2023-03-18T03:18:05.409795628+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2023-03-18T03:18:05.410262917+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2023-03-18T03:18:05.410771385+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2023-03-18T03:18:05.411594181+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.412777103+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T03:18:05.414108707+09:00 [INFO][UDM][SDM] Handle GetNssai
2023-03-18T03:18:05.415007437+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T03:18:05.415536352+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2023-03-18T03:18:05.416009707+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:18:05.416542614+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2023-03-18T03:18:05.417533952+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.418872179+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T03:18:05.420325687+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2023-03-18T03:18:05.420693121+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2023-03-18T03:18:05.42145532+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2023-03-18T03:18:05.422497373+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2023-03-18T03:18:05.4228599+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2023-03-18T03:18:05.423703833+09:00 [INFO][UDM][SDM] Handle GetAmData
2023-03-18T03:18:05.424103351+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T03:18:05.424625697+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:18:05.424974833+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:18:05.425722983+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2023-03-18T03:18:05.426565784+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2023-03-18T03:18:05.427178328+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2023-03-18T03:18:05.427818862+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:18:05.428329911+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2023-03-18T03:18:05.428916443+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2023-03-18T03:18:05.429450573+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2023-03-18T03:18:05.429944304+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2023-03-18T03:18:05.430452854+09:00 [INFO][UDM][SDM] Handle Subscribe
2023-03-18T03:18:05.431275891+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2023-03-18T03:18:05.431565504+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2023-03-18T03:18:05.432109739+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2023-03-18T03:18:05.433162362+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.434561694+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2023-03-18T03:18:05.43624283+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2023-03-18T03:18:05.436987357+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.437673535+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2023-03-18T03:18:05.438575778+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2023-03-18T03:18:05.439060192+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2023-03-18T03:18:05.440197617+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.441862636+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2023-03-18T03:18:05.443168984+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2023-03-18T03:18:05.44327683+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2023-03-18T03:18:05.443346175+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2023-03-18T03:18:05.443811096+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2023-03-18T03:18:05.444125828+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2023-03-18T03:18:05.444222133+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2023-03-18T03:18:05.446011772+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Handle Initial Context Setup Response
2023-03-18T03:18:05.649492179+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2023-03-18T03:18:05.6510405+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2023-03-18T03:18:05.65363265+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T03:18:05+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-03-18T03:18:05.654736589+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2023-03-18T03:18:05.654952383+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2023-03-18T03:18:05.655161663+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2023-03-18T03:18:05.656286422+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.657915877+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2023-03-18T03:18:05.6599137+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2023-03-18T03:18:05.660483722+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=1dda1053-d3f5-46c0-8c4e-367c7e14560a&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2023-03-18T03:18:05.661934816+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.66389811+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:18:05.668334935+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2023-03-18T03:18:05.66902799+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2023-03-18T03:18:05.669756728+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.671054485+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2023-03-18T03:18:05.671595053+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2023-03-18T03:18:05.671910933+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2023-03-18T03:18:05.672094236+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2023-03-18T03:18:05.672266902+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.60.0.1]
2023-03-18T03:18:05.672987547+09:00 [INFO][UDM][SDM] Handle GetSmData
2023-03-18T03:18:05.673236395+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2023-03-18T03:18:05.674105903+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2023-03-18T03:18:05.675165254+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T03:18:05.675784336+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T03:18:05.676650029+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2023-03-18T03:18:05+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0003f0ae0 0xc0003f0b00]
2023-03-18T03:18:05.677238531+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2023-03-18T03:18:05.677465543+09:00 [INFO][SMF][GSM] &{[0xc0003f0ae0 0xc0003f0b00]}
2023-03-18T03:18:05.677652064+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2023-03-18T03:18:05.677848755+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2023-03-18T03:18:05.678625382+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.680031759+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2023-03-18T03:18:05.681652808+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2023-03-18T03:18:05.68249367+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdSmDataGet
2023-03-18T03:18:05.683946381+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T03:18:05.686143826+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2023-03-18T03:18:05.687707411+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2023-03-18T03:18:05.688527899+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:18:05.69003677+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=1dda1053-d3f5-46c0-8c4e-367c7e14560a&target-nf-type=AMF |
2023-03-18T03:18:05.690601155+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2023-03-18T03:18:05.691117241+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2023-03-18T03:18:05.691526438+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2023-03-18T03:18:05.692289313+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2023-03-18T03:18:05.69490338+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2023-03-18T03:18:05.696771594+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2023-03-18T03:18:05.697062299+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2023-03-18T03:18:05.697971451+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2023-03-18T03:18:05.701179601+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:1] Handle PDU Session Resource Setup Response
2023-03-18T03:18:05.702310937+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2023-03-18T03:18:05.702866036+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2023-03-18T03:18:05.703362953+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2023-03-18T03:18:05.705176425+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2023-03-18T03:18:05.705532236+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:67893a27-7574-45fe-a54f-35f9c5aa9772/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2023-03-18T03:15:03.516317415+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleAssociationSetupRequest
2023-03-18T03:15:03.516376138+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142] New node
2023-03-18T03:18:05.695640451+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionEstablishmentRequest
2023-03-18T03:18:05.695727926+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142][CPSEID:0x1][UPSEID:0x1] New session
2023-03-18T03:18:05.707358622+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
37: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::368:af70:5b91:8041/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h3 id="run_sd2">Run UERANSIM (UE[SST:1, SD:0x000002] connects to U-Plane2)</h3>

Additionally, UE connects to U-Plane2 based on SST:1 and SD:0x000002.

```
# ./nr-ue -c ../config/free5gc-ue-sd2.yaml 
UERANSIM v3.2.6
[2023-03-18 03:22:37.989] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 03:22:37.990] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 03:22:37.991] [nas] [info] Selected plmn[001/01]
[2023-03-18 03:22:37.992] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 03:22:37.992] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 03:22:37.992] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 03:22:37.992] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 03:22:37.994] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 03:22:37.995] [nas] [debug] Sending Initial Registration
[2023-03-18 03:22:37.995] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 03:22:37.995] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 03:22:37.995] [rrc] [info] RRC connection established
[2023-03-18 03:22:37.996] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 03:22:37.996] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 03:22:38.015] [nas] [debug] Authentication Request received
[2023-03-18 03:22:38.025] [nas] [debug] Security Mode Command received
[2023-03-18 03:22:38.025] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 03:22:38.058] [nas] [debug] Registration accept received
[2023-03-18 03:22:38.058] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 03:22:38.058] [nas] [debug] Sending Registration Complete
[2023-03-18 03:22:38.058] [nas] [info] Initial Registration is successful
[2023-03-18 03:22:38.058] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 03:22:38.059] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 03:22:38.306] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 03:22:38.311] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 03:22:38.336] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun1, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T03:22:37.992092553+09:00 [INFO][AMF][NGAP][192.168.0.131:40266] Handle Initial UE Message
2023-03-18T03:22:37+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2023-03-18T03:22:37.992680422+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Handle Registration Request
2023-03-18T03:22:37+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2023-03-18T03:22:37.992718521+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Authentication procedure
2023-03-18T03:22:37.993605381+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:37.994546896+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2023-03-18T03:22:37.995640997+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2023-03-18T03:22:37.995925974+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2023-03-18T03:22:37.99674707+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:37.997845061+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2023-03-18T03:22:37.998821773+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2023-03-18T03:22:37.999193187+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0002 0 0 0000000000]
2023-03-18T03:22:37.999401994+09:00 [INFO][UDM][Suci] scheme 0
2023-03-18T03:22:37.999557555+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2023-03-18T03:22:38.002108884+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2023-03-18T03:22:38.003250427+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T03:22:38.003910452+09:00 [INFO][UDM][UEAU] Nil Op
2023-03-18T03:22:38.004557245+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2023-03-18T03:22:38.007077648+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T03:22:38.007639841+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0002-0-0-0000000000/security-information/generate-auth-data |
2023-03-18T03:22:38.00816858+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0002-0-0-0000000000, imsi-001010000000000) to map.
2023-03-18T03:22:38.008448562+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2023-03-18T03:22:38.008510434+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 6536386265626664383564386466653432626534353461363836343132383337
2023-03-18T03:22:38.00873839+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2023-03-18T03:22:38.009441691+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Send Authentication Request
2023-03-18T03:22:38.009562351+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Send Downlink Nas Transport
2023-03-18T03:22:38.011643741+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2023-03-18T03:22:38.012216723+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Handle Authentication Response
2023-03-18T03:22:38.013242784+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2023-03-18T03:22:38.013391524+09:00 [INFO][AUSF][5gAkaAuth] res*: 6536386265626664383564386466653432626534353461363836343132383337
Xres*: 6536386265626664383564386466653432626534353461363836343132383337
2023-03-18T03:22:38.013492945+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2023-03-18T03:22:38.014340549+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2023-03-18T03:22:38.015357796+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2023-03-18T03:22:38.016607993+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2023-03-18T03:22:38.016938964+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2023-03-18T03:22:38.017431253+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0002-0-0-0000000000/5g-aka-confirmation |
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2023-03-18T03:22:38.018414473+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Send Security Mode Command
2023-03-18T03:22:38.018641547+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Send Downlink Nas Transport
2023-03-18T03:22:38.02318563+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2023-03-18T03:22:38.024021105+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle Security Mode Complete
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2023-03-18T03:22:38.02451486+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle InitialRegistration
2023-03-18T03:22:38.025355054+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.026715524+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T03:22:38.027795833+09:00 [INFO][UDM][SDM] Handle GetNssai
2023-03-18T03:22:38.028660967+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T03:22:38.029353539+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2023-03-18T03:22:38.029778578+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:22:38.030496768+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000002}, HomeSnssai: <nil>
2023-03-18T03:22:38.03135045+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.032714862+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T03:22:38.0342453+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2023-03-18T03:22:38.034395505+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2023-03-18T03:22:38.034871088+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2023-03-18T03:22:38.036139112+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2023-03-18T03:22:38.036607834+09:00 [ERRO][UDM][HTTP] unsupported scheme[]
2023-03-18T03:22:38.036863917+09:00 [INFO][UDM][GIN] | 204 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2023-03-18T03:22:38.037613763+09:00 [INFO][UDM][SDM] Handle GetAmData
2023-03-18T03:22:38.037915438+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T03:22:38.038562631+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:22:38.039081055+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:22:38.039545623+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2023-03-18T03:22:38.040224762+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2023-03-18T03:22:38.040974341+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2023-03-18T03:22:38.041506412+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:22:38.041996512+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2023-03-18T03:22:38.042348932+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2023-03-18T03:22:38.042762301+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2023-03-18T03:22:38.043220202+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2023-03-18T03:22:38.043777508+09:00 [INFO][UDM][SDM] Handle Subscribe
2023-03-18T03:22:38.04413706+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2023-03-18T03:22:38.044244657+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2023-03-18T03:22:38.04461863+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2023-03-18T03:22:38.046504746+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.047983881+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2023-03-18T03:22:38.049207256+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2023-03-18T03:22:38.049792916+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2023-03-18T03:22:38.050378627+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2023-03-18T03:22:38.050617084+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2023-03-18T03:22:38.051180669+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Send Registration Accept
2023-03-18T03:22:38.051560263+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Send Initial Context Setup Request
2023-03-18T03:22:38.053568172+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Handle Initial Context Setup Response
2023-03-18T03:22:38.257519087+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2023-03-18T03:22:38.258543093+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle Registration Complete
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2023-03-18T03:22:38.260825824+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T03:22:38+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-03-18T03:22:38.261666649+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle UL NAS Transport
2023-03-18T03:22:38.261924173+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2023-03-18T03:22:38.262151704+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000002}, dnn: internet]
2023-03-18T03:22:38.263123538+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.264836369+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2023-03-18T03:22:38.266728324+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2023-03-18T03:22:38.267198655+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=1dda1053-d3f5-46c0-8c4e-367c7e14560a&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2023-03-18T03:22:38.268626232+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.270604892+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T03:22:38.27490675+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2023-03-18T03:22:38.275630712+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2023-03-18T03:22:38.276419644+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.277682943+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2023-03-18T03:22:38.278225286+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2023-03-18T03:22:38.278534024+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2023-03-18T03:22:38.27877005+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2023-03-18T03:22:38.278923952+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.61.0.1]
2023-03-18T03:22:38.279643215+09:00 [INFO][UDM][SDM] Handle GetSmData
2023-03-18T03:22:38.279893794+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000002"}]
2023-03-18T03:22:38.280664485+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2023-03-18T03:22:38.281592813+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2023-03-18T03:22:38.282205785+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2023-03-18T03:22:38.283093291+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2023-03-18T03:22:38+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0003f0ae0 0xc0003f0b00]
2023-03-18T03:22:38.283657576+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2023-03-18T03:22:38.283864178+09:00 [INFO][SMF][GSM] &{[0xc0003f0ae0 0xc0003f0b00]}
2023-03-18T03:22:38.284036325+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2023-03-18T03:22:38.284225772+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2023-03-18T03:22:38.284982491+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.286344474+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2023-03-18T03:22:38.287701546+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2023-03-18T03:22:38.288348812+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2023-03-18T03:22:38.289561692+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2023-03-18T03:22:38.290362019+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T03:22:38.291770925+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=1dda1053-d3f5-46c0-8c4e-367c7e14560a&target-nf-type=AMF |
2023-03-18T03:22:38.292364579+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2023-03-18T03:22:38.292873092+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2023-03-18T03:22:38.293337025+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2023-03-18T03:22:38.294013539+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2023-03-18T03:22:38.296629164+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2023-03-18T03:22:38.298349515+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2023-03-18T03:22:38.298633044+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Send PDU Session Resource Setup Request
2023-03-18T03:22:38.29952181+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2023-03-18T03:22:38.302169806+09:00 [INFO][AMF][NGAP][192.168.0.131:40266][AMF_UE_NGAP_ID:2] Handle PDU Session Resource Setup Response
2023-03-18T03:22:38.303051289+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2023-03-18T03:22:38.303594285+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2023-03-18T03:22:38.304048258+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2023-03-18T03:22:38.305465428+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2023-03-18T03:22:38.305796283+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:10f23a8b-6283-466f-a905-6a38db8a04b0/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2023-03-18T03:22:38.299341864+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionEstablishmentRequest
2023-03-18T03:22:38.299409120+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805][CPNodeID:192.168.0.143][CPSEID:0x1][UPSEID:0x1] New session
2023-03-18T03:22:38.309892987+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun1` is created as follows.
```
# ip addr show
...
38: uesimtun1: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun1
       valid_lft forever preferred_lft forever
    inet6 fe80::8a09:843c:6be2:4f69/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h3 id="ping_sd1">Ping google.com going through DN=10.60.0.0/16 on U-Plane1 via uesimtun0</h3>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (216.58.220.142) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 216.58.220.142: icmp_seq=1 ttl=61 time=18.8 ms
64 bytes from 216.58.220.142: icmp_seq=2 ttl=61 time=18.3 ms
64 bytes from 216.58.220.142: icmp_seq=3 ttl=61 time=17.8 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
03:31:56.678439 IP 10.60.0.1 > 216.58.220.142: ICMP echo request, id 55, seq 1, length 64
03:31:56.695460 IP 216.58.220.142 > 10.60.0.1: ICMP echo reply, id 55, seq 1, length 64
03:31:57.679557 IP 10.60.0.1 > 216.58.220.142: ICMP echo request, id 55, seq 2, length 64
03:31:57.695994 IP 216.58.220.142 > 10.60.0.1: ICMP echo reply, id 55, seq 2, length 64
03:31:58.680524 IP 10.60.0.1 > 216.58.220.142: ICMP echo request, id 55, seq 3, length 64
03:31:58.696576 IP 216.58.220.142 > 10.60.0.1: ICMP echo reply, id 55, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2.**

<h3 id="ping_sd2">Ping google.com going through DN=10.61.0.0/16 on U-Plane2 via uesimtun1</h3>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun1 -n
PING google.com (142.250.196.142) from 10.61.0.1 uesimtun1: 56(84) bytes of data.
64 bytes from 142.250.196.142: icmp_seq=1 ttl=61 time=20.8 ms
64 bytes from 142.250.196.142: icmp_seq=2 ttl=61 time=18.4 ms
64 bytes from 142.250.196.142: icmp_seq=3 ttl=61 time=18.2 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
03:32:53.278999 IP 10.61.0.1 > 142.250.196.142: ICMP echo request, id 56, seq 1, length 64
03:32:53.298043 IP 142.250.196.142 > 10.61.0.1: ICMP echo reply, id 56, seq 1, length 64
03:32:54.280391 IP 10.61.0.1 > 142.250.196.142: ICMP echo request, id 56, seq 2, length 64
03:32:54.296945 IP 142.250.196.142 > 10.61.0.1: ICMP echo reply, id 56, seq 2, length 64
03:32:55.281406 IP 10.61.0.1 > 142.250.196.142: ICMP echo request, id 56, seq 3, length 64
03:32:55.297757 IP 142.250.196.142 > 10.61.0.1: ICMP echo reply, id 56, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2023.03.17] Updated to free5GC v3.2.1 (2023.02.12) and UERANSIM v3.2.6 (2023.03.17).
- [2022.08.12] Initial release.

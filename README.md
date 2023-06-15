# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - Select UPF based on S-NSSAI
This describes a very simple configuration that uses free5GC and UERANSIM to select the UPF based on S-NSSAI.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/free5gc_ueransim_sample_config)
2. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/free5gc_ueransim_nearby_upf_sample_config)
3. Select UPF based on S-NSSAI (this article)
4. [ULCL(Uplink Classifier)](https://github.com/s5uishida/free5gc_ueransim_ulcl_sample_config)
5. [ULCL with one I-UPF and two PSA-UPFs](https://github.com/s5uishida/free5gc_ueransim_ulcl_2_sample_config)
6. [VPP-UPF with DPDK](https://github.com/s5uishida/free5gc_ueransim_vpp_upf_dpdk_sample_config)

---

<h2 id="misc">Miscellaneous Notes</h2>

- [Install MongoDB 6.0 and free5GC WebUI](https://github.com/s5uishida/free5gc_install_mongodb6_webui)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)

---

<h2 id="toc">Table of Contents</h2>

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
| 10.61.0.0/16 | SST:1 <br> SD:0x000002 | upfgtp | internet | uesimtun0 | U-Plane2 |

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

<h4 id="changes_ue_sd1">Changes in configuration files of UE[SST:1, SD:0x000001] (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-sd1.yaml`
```diff
--- free5gc-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ free5gc-ue-sd1.yaml 2023-03-18 19:15:40.451572298 +0900
@@ -1,9 +1,9 @@
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
 routingIndicator: '0000'
 
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

<h4 id="changes_ue_sd2">Changes in configuration files of UE[SST:1, SD:0x000002] (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-sd2.yaml`
```diff
--- free5gc-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ free5gc-ue-sd2.yaml 2023-03-18 19:15:45.425602882 +0900
@@ -1,9 +1,9 @@
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
 routingIndicator: '0000'
 
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
[2023-03-18 19:25:17.730] [sctp] [info] Trying to establish SCTP connection... (192.168.0.141:38412)
[2023-03-18 19:25:17.733] [sctp] [info] SCTP connection established (192.168.0.141:38412)
[2023-03-18 19:25:17.733] [sctp] [debug] SCTP association setup ascId[7]
[2023-03-18 19:25:17.733] [ngap] [debug] Sending NG Setup Request
[2023-03-18 19:25:17.740] [ngap] [debug] NG Setup Response received
[2023-03-18 19:25:17.741] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T19:25:17.734605363+09:00 [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.131:50228
2023-03-18T19:25:17.735510262+09:00 [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.131:50228
2023-03-18T19:25:17.739838968+09:00 [INFO][AMF][NGAP][192.168.0.131:50228] Handle NG Setup request
2023-03-18T19:25:17.740052244+09:00 [INFO][AMF][NGAP][192.168.0.131:50228] Send NG-Setup response
```

<h3 id="run_sd1">Run UERANSIM (UE[SST:1, SD:0x000001])</h3>

Confirm that the packet goes through the DN of U-Plane1 based on SST:1 and SD:0x000001.

<h4 id="con_sd1">UE connects to U-Plane1 based on SST:1 and SD:0x000001</h4>

```
# ./nr-ue -c ../config/free5gc-ue-sd1.yaml 
UERANSIM v3.2.6
[2023-03-18 19:27:29.241] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 19:27:29.242] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 19:27:29.242] [nas] [info] Selected plmn[001/01]
[2023-03-18 19:27:29.242] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 19:27:29.243] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 19:27:29.243] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 19:27:29.243] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 19:27:29.245] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 19:27:29.245] [nas] [debug] Sending Initial Registration
[2023-03-18 19:27:29.245] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 19:27:29.246] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 19:27:29.246] [rrc] [info] RRC connection established
[2023-03-18 19:27:29.246] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 19:27:29.247] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 19:27:29.284] [nas] [debug] Authentication Request received
[2023-03-18 19:27:29.295] [nas] [debug] Security Mode Command received
[2023-03-18 19:27:29.296] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 19:27:29.346] [nas] [debug] Registration accept received
[2023-03-18 19:27:29.346] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 19:27:29.347] [nas] [debug] Sending Registration Complete
[2023-03-18 19:27:29.347] [nas] [info] Initial Registration is successful
[2023-03-18 19:27:29.347] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 19:27:29.347] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 19:27:29.608] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 19:27:29.613] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 19:27:29.634] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T19:27:29.244300594+09:00 [INFO][AMF][NGAP][192.168.0.131:50228] Handle Initial UE Message
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2023-03-18T19:27:29.247563579+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2023-03-18T19:27:29.24804485+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2023-03-18T19:27:29.24900818+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.250375214+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2023-03-18T19:27:29.25232073+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2023-03-18T19:27:29.252547212+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2023-03-18T19:27:29.253594061+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.255076653+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2023-03-18T19:27:29.260195762+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2023-03-18T19:27:29.260431813+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2023-03-18T19:27:29.260627769+09:00 [INFO][UDM][Suci] scheme 0
2023-03-18T19:27:29.260839634+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2023-03-18T19:27:29.261772533+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.262761619+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2023-03-18T19:27:29.265271702+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2023-03-18T19:27:29.268600122+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T19:27:29.270342467+09:00 [INFO][UDM][UEAU] Nil Op
2023-03-18T19:27:29.273492808+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2023-03-18T19:27:29.275467894+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T19:27:29.276942686+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2023-03-18T19:27:29.277701339+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2023-03-18T19:27:29.277931327+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2023-03-18T19:27:29.278129468+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 3164343136666632383263663730653262383239396138613862343339333666
2023-03-18T19:27:29.278410541+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2023-03-18T19:27:29.278936696+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2023-03-18T19:27:29.279978333+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2023-03-18T19:27:29.282067135+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2023-03-18T19:27:29.282502718+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2023-03-18T19:27:29.283368872+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2023-03-18T19:27:29.283638203+09:00 [INFO][AUSF][5gAkaAuth] res*: 3164343136666632383263663730653262383239396138613862343339333666
Xres*: 3164343136666632383263663730653262383239396138613862343339333666
2023-03-18T19:27:29.284317824+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2023-03-18T19:27:29.285977967+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2023-03-18T19:27:29.286869692+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2023-03-18T19:27:29.289682948+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2023-03-18T19:27:29.290029685+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2023-03-18T19:27:29.290547071+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2023-03-18T19:27:29.291247281+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2023-03-18T19:27:29.291317593+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2023-03-18T19:27:29.29343784+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2023-03-18T19:27:29.293981246+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2023-03-18T19:27:29.294354412+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2023-03-18T19:27:29.295095072+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.296261048+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T19:27:29.297361589+09:00 [INFO][UDM][SDM] Handle GetNssai
2023-03-18T19:27:29.298255596+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T19:27:29.300354313+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2023-03-18T19:27:29.300763907+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:27:29.301551202+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2023-03-18T19:27:29.302417959+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.303502519+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T19:27:29.305025568+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2023-03-18T19:27:29.305243367+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2023-03-18T19:27:29.306149637+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2023-03-18T19:27:29.308549802+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2023-03-18T19:27:29.308912482+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2023-03-18T19:27:29.309547134+09:00 [INFO][UDM][SDM] Handle GetAmData
2023-03-18T19:27:29.309939486+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T19:27:29.310537844+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:27:29.310974907+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:27:29.31162448+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2023-03-18T19:27:29.312316781+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2023-03-18T19:27:29.314444909+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2023-03-18T19:27:29.31489103+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:27:29.315448192+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2023-03-18T19:27:29.316203297+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2023-03-18T19:27:29.316767422+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2023-03-18T19:27:29.31728885+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2023-03-18T19:27:29.317979961+09:00 [INFO][UDM][SDM] Handle Subscribe
2023-03-18T19:27:29.31879871+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2023-03-18T19:27:29.319149681+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2023-03-18T19:27:29.319587522+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2023-03-18T19:27:29.320528555+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.321938211+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2023-03-18T19:27:29.324874623+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2023-03-18T19:27:29.325429528+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.326378133+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2023-03-18T19:27:29.327320534+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2023-03-18T19:27:29.329436973+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2023-03-18T19:27:29.334073351+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.335560275+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2023-03-18T19:27:29.339486842+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2023-03-18T19:27:29.339567445+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2023-03-18T19:27:29.339626976+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2023-03-18T19:27:29.340333105+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2023-03-18T19:27:29.340728705+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2023-03-18T19:27:29.341027104+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2023-03-18T19:27:29.343410679+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Handle Initial Context Setup Response
2023-03-18T19:27:29.549732528+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2023-03-18T19:27:29.550550008+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2023-03-18T19:27:29.551994322+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2023-03-18T19:27:29+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-03-18T19:27:29.552493813+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2023-03-18T19:27:29.552702204+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2023-03-18T19:27:29.552968914+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2023-03-18T19:27:29.553982615+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.555224762+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2023-03-18T19:27:29.559002413+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2023-03-18T19:27:29.559331547+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=413b9ee2-a743-4a3b-b131-4e50c653b214&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2023-03-18T19:27:29.560595116+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.562313545+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:27:29.565140225+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2023-03-18T19:27:29.566725135+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2023-03-18T19:27:29.569572937+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.570938197+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2023-03-18T19:27:29.571537188+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2023-03-18T19:27:29.571730041+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2023-03-18T19:27:29.571924057+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2023-03-18T19:27:29.572112415+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.60.0.1]
2023-03-18T19:27:29.572797416+09:00 [INFO][UDM][SDM] Handle GetSmData
2023-03-18T19:27:29.573115934+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2023-03-18T19:27:29.573893001+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2023-03-18T19:27:29.576359137+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T19:27:29.576875271+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T19:27:29.577523988+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2023-03-18T19:27:29+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0003f2240 0xc0003f2260]
2023-03-18T19:27:29.577932187+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2023-03-18T19:27:29.578146075+09:00 [INFO][SMF][GSM] &{[0xc0003f2240 0xc0003f2260]}
2023-03-18T19:27:29.578507943+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2023-03-18T19:27:29.578677119+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2023-03-18T19:27:29.579459238+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.580646536+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2023-03-18T19:27:29.582479049+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2023-03-18T19:27:29.583355667+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdSmDataGet
2023-03-18T19:27:29.586099694+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-03-18T19:27:29.588167881+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2023-03-18T19:27:29.589103549+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2023-03-18T19:27:29.589921695+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:27:29.591198797+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=413b9ee2-a743-4a3b-b131-4e50c653b214&target-nf-type=AMF |
2023-03-18T19:27:29.591760335+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2023-03-18T19:27:29.592218511+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2023-03-18T19:27:29.593158836+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2023-03-18T19:27:29.593777923+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2023-03-18T19:27:29.596323872+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2023-03-18T19:27:29.600991999+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2023-03-18T19:27:29.60117533+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2023-03-18T19:27:29.60202873+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2023-03-18T19:27:29.604959429+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:1] Handle PDU Session Resource Setup Response
2023-03-18T19:27:29.606116142+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2023-03-18T19:27:29.606647635+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2023-03-18T19:27:29.607102535+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2023-03-18T19:27:29.608643096+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2023-03-18T19:27:29.608976227+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:73804519-1236-4991-90db-e1b82c8dfc28/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2023-03-18T19:27:29.593228827+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionEstablishmentRequest
2023-03-18T19:27:29.593262784+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142][CPSEID:0x1][UPSEID:0x1] New session
2023-03-18T19:27:29.607060430+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
10: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::3873:9adb:55fb:719d/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_sd1">Ping google.com going through DN=10.60.0.0/16 on U-Plane1</h4>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.175.110) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.175.110: icmp_seq=1 ttl=61 time=21.3 ms
64 bytes from 172.217.175.110: icmp_seq=2 ttl=61 time=18.1 ms
64 bytes from 172.217.175.110: icmp_seq=3 ttl=61 time=17.5 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
19:31:34.929800 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 10, seq 1, length 64
19:31:34.948826 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 10, seq 1, length 64
19:31:35.930266 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 10, seq 2, length 64
19:31:35.946545 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 10, seq 2, length 64
19:31:36.931389 IP 10.60.0.1 > 172.217.175.110: ICMP echo request, id 10, seq 3, length 64
19:31:36.947230 IP 172.217.175.110 > 10.60.0.1: ICMP echo reply, id 10, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2.**

<h3 id="run_sd2">Run UERANSIM (UE[SST:1, SD:0x000002])</h3>

Then the UE disconnects from gNodeB and connects to gNodeB using the configuration file for SST:1 and SD:0x000002.
Confirm that the packet goes through the DN of U-Plane2 based on SST:1 and SD:0x000002.

<h4 id="con_sd2">UE connects to U-Plane2 based on SST:1 and SD:0x000002</h4>

```
# ./nr-ue -c ../config/free5gc-ue-sd2.yaml 
UERANSIM v3.2.6
[2023-03-18 19:34:17.464] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 19:34:17.464] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 19:34:17.464] [nas] [info] Selected plmn[001/01]
[2023-03-18 19:34:17.465] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 19:34:17.465] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 19:34:17.465] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 19:34:17.465] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 19:34:17.467] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 19:34:17.467] [nas] [debug] Sending Initial Registration
[2023-03-18 19:34:17.467] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 19:34:17.468] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 19:34:17.468] [rrc] [info] RRC connection established
[2023-03-18 19:34:17.468] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 19:34:17.468] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 19:34:17.487] [nas] [debug] Authentication Request received
[2023-03-18 19:34:17.495] [nas] [debug] Security Mode Command received
[2023-03-18 19:34:17.495] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 19:34:17.523] [nas] [debug] Registration accept received
[2023-03-18 19:34:17.523] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 19:34:17.523] [nas] [debug] Sending Registration Complete
[2023-03-18 19:34:17.523] [nas] [info] Initial Registration is successful
[2023-03-18 19:34:17.523] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 19:34:17.524] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 19:34:17.773] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 19:34:17.776] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 19:34:17.797] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2023-03-18T19:34:17.465970994+09:00 [INFO][AMF][NGAP][192.168.0.131:50228] Handle Initial UE Message
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2023-03-18T19:34:17.466503058+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Handle Registration Request
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2023-03-18T19:34:17.466540555+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Authentication procedure
2023-03-18T19:34:17.467378668+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.468464994+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2023-03-18T19:34:17.469527825+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2023-03-18T19:34:17.469763825+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2023-03-18T19:34:17.470526952+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.472389584+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2023-03-18T19:34:17.473899294+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2023-03-18T19:34:17.474261654+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2023-03-18T19:34:17.474773476+09:00 [INFO][UDM][Suci] scheme 0
2023-03-18T19:34:17.475030659+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2023-03-18T19:34:17.475945415+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2023-03-18T19:34:17.47719919+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T19:34:17.477864472+09:00 [INFO][UDM][UEAU] Nil Op
2023-03-18T19:34:17.478386458+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2023-03-18T19:34:17.48106833+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-03-18T19:34:17.481505688+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2023-03-18T19:34:17.482011258+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2023-03-18T19:34:17.482168311+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2023-03-18T19:34:17.482244139+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 3430376332343766363762383833356137623531396634373438393663343665
2023-03-18T19:34:17.482395969+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2023-03-18T19:34:17.482963795+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Send Authentication Request
2023-03-18T19:34:17.483031459+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Send Downlink Nas Transport
2023-03-18T19:34:17.484932093+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2023-03-18T19:34:17.485095074+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2] Handle Authentication Response
2023-03-18T19:34:17.485912427+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2023-03-18T19:34:17.486135046+09:00 [INFO][AUSF][5gAkaAuth] res*: 3430376332343766363762383833356137623531396634373438393663343665
Xres*: 3430376332343766363762383833356137623531396634373438393663343665
2023-03-18T19:34:17.48679899+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2023-03-18T19:34:17.487544877+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2023-03-18T19:34:17.488342313+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2023-03-18T19:34:17.489228961+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2023-03-18T19:34:17.489620607+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2023-03-18T19:34:17.490095403+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2023-03-18T19:34:17.490561865+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Send Security Mode Command
2023-03-18T19:34:17.490619018+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Send Downlink Nas Transport
2023-03-18T19:34:17.492835813+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2023-03-18T19:34:17.493225177+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle Security Mode Complete
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2023-03-18T19:34:17.493641221+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle InitialRegistration
2023-03-18T19:34:17.494357911+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.495431911+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T19:34:17.496377485+09:00 [INFO][UDM][SDM] Handle GetNssai
2023-03-18T19:34:17.497388676+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T19:34:17.497941793+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2023-03-18T19:34:17.498392885+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:34:17.498889015+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000002}, HomeSnssai: <nil>
2023-03-18T19:34:17.499511155+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.500609545+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-03-18T19:34:17.502001834+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2023-03-18T19:34:17.502201657+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2023-03-18T19:34:17.503153891+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2023-03-18T19:34:17.504101866+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2023-03-18T19:34:17.504481343+09:00 [ERRO][UDM][HTTP] unsupported scheme[]
2023-03-18T19:34:17.504760951+09:00 [INFO][UDM][GIN] | 204 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2023-03-18T19:34:17.505171265+09:00 [INFO][UDM][SDM] Handle GetAmData
2023-03-18T19:34:17.505568236+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2023-03-18T19:34:17.506034859+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:34:17.506421856+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:34:17.507025337+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2023-03-18T19:34:17.50759439+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2023-03-18T19:34:17.508250515+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2023-03-18T19:34:17.508635594+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:34:17.509197656+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2023-03-18T19:34:17.509496477+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2023-03-18T19:34:17.510038671+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2023-03-18T19:34:17.510360156+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2023-03-18T19:34:17.510957796+09:00 [INFO][UDM][SDM] Handle Subscribe
2023-03-18T19:34:17.511431134+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2023-03-18T19:34:17.511677199+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2023-03-18T19:34:17.512158388+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2023-03-18T19:34:17.51304977+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.514365353+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2023-03-18T19:34:17.515315935+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2023-03-18T19:34:17.516288472+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2023-03-18T19:34:17.516886575+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2023-03-18T19:34:17.517234622+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2023-03-18T19:34:17.517455566+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Send Registration Accept
2023-03-18T19:34:17.517548588+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Send Initial Context Setup Request
2023-03-18T19:34:17.519663316+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Handle Initial Context Setup Response
2023-03-18T19:34:17.726606186+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2023-03-18T19:34:17.727998545+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle Registration Complete
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2023-03-18T19:34:17.730033596+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Uplink NAS Transport (RAN UE NGAP ID: 2)
2023-03-18T19:34:17+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-03-18T19:34:17.730999121+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Handle UL NAS Transport
2023-03-18T19:34:17.731219519+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2023-03-18T19:34:17.73143293+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000002}, dnn: internet]
2023-03-18T19:34:17.732395963+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.733978282+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2023-03-18T19:34:17.736255181+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2023-03-18T19:34:17.736549655+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=413b9ee2-a743-4a3b-b131-4e50c653b214&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2023-03-18T19:34:17.73797035+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.739612446+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-03-18T19:34:17.743871934+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2023-03-18T19:34:17.744573566+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2023-03-18T19:34:17.745355766+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.746653521+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2023-03-18T19:34:17.747205595+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2023-03-18T19:34:17.747428898+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2023-03-18T19:34:17.747647132+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2023-03-18T19:34:17.747845341+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.61.0.1]
2023-03-18T19:34:17.748621598+09:00 [INFO][UDM][SDM] Handle GetSmData
2023-03-18T19:34:17.748963839+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000002"}]
2023-03-18T19:34:17.749789806+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2023-03-18T19:34:17.750646296+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2023-03-18T19:34:17.75118592+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000002%22%7D |
2023-03-18T19:34:17.751880707+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2023-03-18T19:34:17+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0003f2ae0 0xc0003f2b00]
2023-03-18T19:34:17.75222602+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2023-03-18T19:34:17.752519233+09:00 [INFO][SMF][GSM] &{[0xc0003f2ae0 0xc0003f2b00]}
2023-03-18T19:34:17.752684906+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2023-03-18T19:34:17.752951193+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2023-03-18T19:34:17.753600796+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.754888499+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2023-03-18T19:34:17.756257975+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2023-03-18T19:34:17.757280818+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2023-03-18T19:34:17.758277967+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2023-03-18T19:34:17.758992605+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2023-03-18T19:34:17.760413121+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=413b9ee2-a743-4a3b-b131-4e50c653b214&target-nf-type=AMF |
2023-03-18T19:34:17.760951903+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2023-03-18T19:34:17.761345674+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2023-03-18T19:34:17.761802473+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2023-03-18T19:34:17.76236535+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:2][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2023-03-18T19:34:17.764984257+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2023-03-18T19:34:17.766497057+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2023-03-18T19:34:17.766786159+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Send PDU Session Resource Setup Request
2023-03-18T19:34:17.767616657+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2023-03-18T19:34:17.770213013+09:00 [INFO][AMF][NGAP][192.168.0.131:50228][AMF_UE_NGAP_ID:2] Handle PDU Session Resource Setup Response
2023-03-18T19:34:17.771057608+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2023-03-18T19:34:17.771475742+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2023-03-18T19:34:17.771880539+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2023-03-18T19:34:17.773282146+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2023-03-18T19:34:17.773503363+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:cf049dc6-ea4d-4e7a-a13c-679b8bb8e8b8/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2023-03-18T19:34:17.774659569+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionEstablishmentRequest
2023-03-18T19:34:17.774693252+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805][CPNodeID:192.168.0.143][CPSEID:0x1][UPSEID:0x1] New session
2023-03-18T19:34:17.784494287+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
11: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::ace7:dd5d:309e:1bc2/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_sd2">Ping google.com going through DN=10.61.0.0/16 on U-Plane2</h4>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.199.110) from 10.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.199.110: icmp_seq=1 ttl=61 time=26.7 ms
64 bytes from 142.250.199.110: icmp_seq=2 ttl=61 time=30.6 ms
64 bytes from 142.250.199.110: icmp_seq=3 ttl=61 time=21.5 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
19:37:43.503241 IP 10.61.0.1 > 142.250.199.110: ICMP echo request, id 11, seq 1, length 64
19:37:43.527884 IP 142.250.199.110 > 10.61.0.1: ICMP echo reply, id 11, seq 1, length 64
19:37:44.503827 IP 10.61.0.1 > 142.250.199.110: ICMP echo request, id 11, seq 2, length 64
19:37:44.532098 IP 142.250.199.110 > 10.61.0.1: ICMP echo reply, id 11, seq 2, length 64
19:37:45.504714 IP 10.61.0.1 > 142.250.199.110: ICMP echo request, id 11, seq 3, length 64
19:37:45.524338 IP 142.250.199.110 > 10.61.0.1: ICMP echo reply, id 11, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF based on S-NSSAI. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2023.03.18] Updated to free5GC v3.2.1 (2023.02.12) and UERANSIM v3.2.6 (2023.03.17).
- [2022.08.12] Initial release.

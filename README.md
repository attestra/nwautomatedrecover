# nwautomatedrecover
Tools for scripted recovers using the Networker REST API.

NOTE: Tested in only one environment, so please test it in a lab before running in production. Also, as for any piece of code that you download from the internet, your should read and understand the code completely before running it. Provided without warranty.
## Overview
This repo contains tools to help Dell/EMC Networker backup administrators automate recoveries
## Usage and examples
Output from `./nwrestvmrecover`  -h
```
nwrestvmrecover [OPTION]... [-x] vmname - Script to trigger a restore via the Networker REST API

where:
   -h  shows this help text
    -a  Authentication digest
    -b  Networker (backup) server name
    -c  Destination cluster resource
    -d  Destination datacenter
    -e  Destination (ESXi) host
    -f  VM Folder
    -j  Job name
    -n  VM name (once recovered, destination)
    -p  Power on after recover?
    -r  Reconnect NIC after recover?
    -s  Destination datastore
    -v  vCenter Server
    -x  Must be present to confirm that the recover must be executed
    vmname : Name of the Virtual Machine to recover (source)

All the parameters are required by this script (but may not be by the API)

Note: Always test in a lab first
```
## Prerequisites/compatibility
### Prerequisites
  - bash
  - jq
#### Values needed for any recover
 - A username/password combo that can perform recoveries via the REST API
   - To generate the authentication digest, you have to Base64 encode your credentials
     - To do so, you can use the https://www.base64encode.org/ site. Encode in the format username:password
 - The name of your Networker server
#### Needed for VM recovers
You have to provide information about your VMWare infrastructure to be able to perform a recovery.

The first information to have it the name of your vCenter server. This one is rather easy.  For the others, you will have to use the MOB (Managed Object Reference) API (should be accessible at https://vcenterserver/mob). See https://kb.vmware.com/s/article/1017126

The idea is to take note of the VMWare objects that will be used for _recovery_ (where will the new VM go):
 - Cluster resource
 - Datacenter
 - Datastore
 - Host
 - VM Folder
 - VM Name

Here's some guidance (there may be another way):
  - Go to https://vcenterserver/mob
  - Click on the second property - content
  - Click on the rootFolder property. For me it is group-d1 (Datacenters)
  - In the childEntity property, you have a list of the MOB value for your datacenters and their respective labels. For exemple, you may see datacenter-33 (YourLabel). So if you want to recover in your _datacenter_ named "YourLabel", you must take note of its MOB property: datacenter-33.
   - Open in a new tab the link of the datacenter (datacenter-33 in our example), and the link of the vmFolder property (bottom)
     - Go to the tab of the datacenter link
      - Click the link in the hostFolder property (group-h*), this will show you a list of your clusters (or standalone host) in the childEntity property
      - Click the link of the _cluster_ where you want to recover the data (domain-****)
      - On that page, you can gather 2 parameters
        - _datastore_
          - Take note of the datastore where you want to recover to. For example: datastore-0000 (MyRecoverDatastore)
          - Creating a dedicated Datastore for recovery purpose is recommended, to avoid filling a production datastore
        - _host_
          - Take note of the value of the host where you want to recover to. For example: host-0000 (MyRecoverDatastore)
     - Go to the tab of the _vmFolder_ property (group-v*)
       - Take note of the value of the folder in which to create the destination VM (group-v*)

### Compatibility
 - Tested with bash 4.1.2 on RHEL 6
 - Tested with Networker 19.1.1/vProxy
## Known issues
 - Currently only for VMWare recoveries
   - The only recovery type supported: New (creates a new VM and recovers data into it)
## References/Documentation
Networker 19.1 REST API Reference Guide: https://www.dellemc.com/pt-pt/collaterals/unauth/quick-reference-guides/products/data-protection/docu94009.pdf
The blog post that gave me the idea of starting to work on automated recoveries: https://nsrd.info/blog/2020/03/30/basics-finding-unprotected-virtual-machines-with-networker/
## Contribute
Feel free to open issues if you experience problems, but make sure you look at the TROUBLESHOOTING.md page before to make sure it's not a Networker or VMWare issue.
Pull requests are welcome, whether it's for documentation, bugfix or enhancement.

---
layout: post
title: "Nutanix Distributed Storage Fabric Concepts" 
excerpt: "Centrally Accessed, Physically Dispersed Storage"
tags: 
- Hyper-V
- Deployment
- ESXi
- Nutanix
image:
  thumb: /nutanix-storage-concepts/nutanix-storage-concepts-00.png
comments: true
date: 2023-04-12T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix Storage and Data Management" src="/images/nutanix-storage-concepts/nutanix-storage-concepts-00.png">
In light of recent [career developments](https://www.linkedin.com/posts/activity-7043559030807494657-i8DL){:target="_blank"}, I'm continuing to deepen my knowledge into Nutanix products and offerings. In this post we will take a closer look into Nutanix storage.

One of the major components and unique selling points of the Nutanix cloud platform is Nutanix Unified Storage offering. The backbone of which is the Nutanix Distributed Storage Fabric (DSF).

Put simply, DSF is a software-defined storage solution that provides a unified storage fabric for all applications, both virtualized and non-virtualized. 

DSF is built on a distributed architecture that allows for the pooling of storage resources across multiple hosts and nodes, providing high performance and scalability: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSF High Level" src="/images/nutanix-storage-concepts/nutanix-storage-concepts-01.png">

DSF simplifies storage management by providing a single interface for provisioning and managing storage resources, making it easier for organizations to deploy and manage their storage infrastructure. Finally, DSF also includes data protection features such as data replication, erasure coding, and data-at-rest encryption. 

In this post let's take a look at the concepts and components that make up the Nutanix Distributed Storage Fabric. 
{% include _toc.html %}
### CVM
As I covered in my [first Nutanix post](/nested-nutanix-ce-deployment/#nutanix---introduction--architecture){:target="_blank"}, A Nutanix Controller VM (CVM) is deployed on each hypervisor participating in a Nutanix cluster. The CVM runs the Nutanix software and serves all of the I/O operations for the hypervisor and all VMs running on that host.

### Storage Pool
A storage pool is the group of storage devices (HDD, SSD) in the cluster. The storage pool can span multiple Nutanix hosts and is expanded as the cluster host count is increased. 
In most deployments a single storage pool is used.

### Storage Container
A storage container is a logical segmentation of the storage pool. Storage containers contain VMs or files (vDisks).
Some configurations options are available at the storage container level such as Replication Factor. These configuration options are applied at the individual VM or file level.
Storage containers typically have direct 1 to 1 mapping with a datastore (NFS in the case of ESXi and SMB in the case of Hyper-V).  

### vDisk
A vDisk is any file stored in a storage container that is over 512KB in size. For example this might be a VMDK file for an ESXi VM virtual disk or a VHD or a Hyper-V VM virtual disk.
vDisks are broken into extents which are grouped and stored on the physical disk as an extent group.

### Extent
An extent is a 1MB piece of of logically contiguous data consisting of a number of contiguous vBlocks. The number of vBlocks in an extent is dependant on the block size set by the guest VM's operating system. Extents are read/modified/written on a sub-extent basis known as a slice. A slice may be trimmed when moving when moving into the cache depending on the amount of data being read/cached.

Extents are dynamically distributed among extent groups to provide data striping across hosts/disks to improve performance.

### Extent Group
An extent group is a 1MB or 4MB piece of contiguous stored data.  This data is stored as a file on a storage device owned by a CVM.

### vBlock
A vBlock is a 1MB piece of vDisk space.  For example a 100MB vDisk will consist of 100 x 1MB vBlocks. vBlock 0 would be for 0-1MB, vBlock 1 would be from 1-2MB etc. As discussed above, vBlocks map to Extents. 

### OpLog
The oplog operates in a way similar to a filesystem journal and is built as a staging area to handle bursts of random writes, consolidate them and then sequentially "drain" the written data to the extent store. For sequential workloads, the OpLog is bypassed and writes go directly to the extent store. 

If data is currently sitting in the OpLog and has not been drained, all read requests will be directly fulfilled from the OpLog until they have been drained, where they would then be served by the extent store/unified cache.

### Extent Store
The Extent Store is the persistent bulk storage of DSF and spans all device tiers (Optane SSD, PCIe SSD, SATA SSD, HDD) and is extensible to facilitate additional devices/tiers. Data entering the extent store is either being drained from the OpLog or is sequential/sustained in nature and has bypassed the OpLog directly. 

The Nutanix Intelligent Lifecycle Manager (ILM) will determine tier placement dynamically based upon I/O patterns, number of accesses of data and weight given to individual tiers and will move data between tiers.

### Autonomous Extent Store
The Autonomous Extent Store (AES) is a different method for writing/storing data in the Extent Store. It leverages a mix of primarily local + global metadata allowing for much more efficient sustained performance due to metadata locality. For sustained random write workloads, these will bypass the OpLog and be written directly to the Extent Store using AES. For bursty random workloads these will take the typical OpLog I/O path then drain to the Extent Store using AES where possible.

### Unified Cache
The Unified Cache is a read cache which is used for data, metadata and deduplication, and is stored in the CVM’s memory. 

Upon a read request of data not in the cache (or based upon a particular fingerprint), the data will be read from the extent store and will also be placed into the single-touch pool of the Unified Cache which completely sits in memory, where it will use LRU (least recently used) until it is evicted from the cache. Any subsequent read request will “move” (no data is actually moved, just cache metadata) the data into the multi-touch pool.

Any read request for data in the multi-touch pool will cause the data to go to the peak of the multi-touch pool where it will be given a new LRU counter.

## Pulling It Together
The following diagram illustrates at a low level how the above elements are interconnected to provide the Nutanix Distributed Storage Fabric: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSF Low Level" src="/images/nutanix-storage-concepts/nutanix-storage-concepts-02.png">
Notes:
1. Block size is determined by the VM guest filesystem<br>
2. Extents or vBlocks 

The following diagram illustrates the I/O path from a guest VM into the DSF:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSF IO Path" src="/images/nutanix-storage-concepts/nutanix-storage-concepts-03.png">

As can be seen from above, DSF is accessed using industry standard storage protocols thus providing a unified storage fabric for all applications, both virtualized and non-virtualized.

That will do it for this time. If you are still thirsty for more take a look at the [Book of AOS](https://www.nutanixbible.com/4c-book-of-aos-dsf.html){:target="_blank"} from the Nutanix Bible.

Next time we will take a look at the data protection features built into DSF such as data replication, erasure coding, and data-at-rest encryption. 

You can find that post here: [Nutanix Distributed Storage Fabric Protection Concepts](/nutanix-storage-protection-concepts/){:target="_blank"}.

-Chris
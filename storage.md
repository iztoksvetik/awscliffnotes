---
layout: page
title: Storage
---

 1. AWS Backup
 1. [Amazon Elastic Block Storage (Amazon EBS)](#amazon-elastic-block-storage)
 1. AWS Elastic Disaster recovery
 1. Amazon Elastic File System (Amazon EFS)
 1. Amazon FSX (for all types)
 1. Amazon S3
 1. Amazon S3 Glacier
 1. AWS Storage Gateway

## General concepts 

 1. [Storage types](#primary-storage-types)

## Amazon Elastic Block Storage

Block storage designed to use with **EC2**. 

 - EBS volumes behave like raw, unformatted block devices. 
 - They persist independently from the life of (EC2) instance
 - They can be moved between instances
 - Configuration can be changed dynamically (e.g. size, type, ...) - **Elastic Volumes**
 - Can scale up to PBs of data
 - Billed for provisioned storage
 - Supports encryption using AWS KMS, can be enabled by default
 - Multi-attach enables attaching ioX volumes to up to 16 Nitro based instances
 - Monitoring trough AWS Management Console and CloudWatch
 - Backups can be managed by AWS Backups and using AWS Organizations can be managed centrally
 - Multiple volumes can be attached to single instance as long as they are all in the same AZ

### Data durability

 - (Automatically) Replicated **within an AZ**
 - 99.8% durability (Annual failure rate of 0.1-0.2%), with **io2** 99.999%
 - EBS Snapshots with automated lifecycle policies to backup to **S3**

### Availability

99.999% availability.

### Block storage types

 - **Instance:** 
   - ephemeral storage (terminated with instance), persists on reboot, lost on termination/hibernation/stopping
   - physically attached to host computer 
   - similar to EBS in initial configuration, functionally resemble a directly attached disk
   - cannot be detached and moved to another instance
 - **Amazon EBS:** 
   - attach to an instance, can be moved between instances
   - volumes are persistent
 - **Snapshots:** 
   - incremental point-in-time copies of the data of EBS volumes
   - used to restore volumes, resize, move between AZs or Regions
   - can use Amazon Data Lifecycle Manager (Amazon DLM) to automate

### Volume types

#### SSD backed volumes

Offers two sub-types, **gp** for general purpose balance price and performance, whereas **st** (provisioned IOPS) are the highest performance IOPS drives for most demanding transactional applications (SAP HANA, Microsoft SQL Server...).

 1. **gp2:** (performance scales with volume size)
   - baseline performance of 3 IOPS per GiB size (min 100IOPS@33.33GiB or less, max 16,000IOPS@5334GiB or more)
   - size: 1GiB - 16TiB
   - single digit ms latency
   - burst up to 3000 IOPS, when baseline higher no burst
   - 3 IOPS per GiB credit, when using less, accumulate for burst
   - flat rate pricing based on volume size
 1. **gp3:** (performance scales independent of volume size)
   - 3000 IOPS and 125 MB/s base performance
   - can provision up to 16,000 IOPS and 1,000 MB/s 
     - max. 500 IOPS per GiB so max from 32GiB
     - max. 0.25 MB/s per IOPS (8GiB or larger volume with 4,000 IOPS = 1,000 MB/s)
   - flat rate pricing for volume size and tiered for IOPS and throughput
 1. **io1:**
   - size: 4GiB - 16TiB
   - 100 - 32,000 IOPS (64,000 on Nitro)
   - max. IO size 256 KiB
   - up to 500 MB/s throughput (with IO size at max, peak at 2,000 IOPS)
   - at more than 32,000 IOPS max I/O size 16KiB, max throughput of 1,000 MB/s
   - 99.8-99.9% durability
   - max ratio IOPS to GiB 50:1
 1. **io2:**
   - most current, recommended for all new deployments
   - same size, IOPS, I/O size, throughput as **io1**
   - 99.999% durability
   - not available for EC2 R5b type
   - max ratio IOPS to GiB 500:1


#### HDD backed volumes

 1. **st1:**
   - frequently accessed throughput intensive workloads
   - 1 MB I/O size
   - baseline 40 MB/s throughput per TB of volume size
   - 5 MB/s @ 125 GB to 500 MB/s @ 12,775 GB
   - burst from 40 MB/s per TiB to 500 MB/s
   - designed to deliver provisioned performance 90% of the time
   - size: 125GiB - 16TiB
 1. **sc1:**
   - less frequently accessed
   - lowest cost Cold HDD
   - 1 MB I/O size
   - 12MB/s per TB
   - 1.5MB/s @ 125GiB to 192MB/s @ 16,384GiB
   - burst of 12MB/s per TB up to 250MB/s
   - designed to deliver provisioned performance 90% of the time
   - size: 125GiB - 16TiB


### Performance

 - **IOPS:**
   - I/O size capping:
     - SSD: 256 KiB (kibibyte)
     - HDD: 1024 KiB
   - When small I/O operations are physically contiguous, EBS attempts to merge into a single I/O op (up to max size)
   - Large sequential I/O ops are divided into separate ops of max size
   - Noncontiguous ops are not merged
 - **Throughput:**
   - SSD volumes with large I/O sizes might have lower than provisioned IOPS if throughput limit is reached
   - HDD volumes with small or random I/O workloads might not reach provisioned throughput as IOPS limit might be reached beforehand
 - **Burst balance:**
   - For some volume types you are able to burst performance above provisioned limit
   - When operating within provisioned baseline you accumulate burst credits
   - When you go above, you use them
   - If the balance is depleted, you can't go above baseline
 - **Latency:**
   - *SSD:* from <1ms to <10ms
   - *HDD:* average of <99ms
   - Number of pending I/O requests (volume queue) can affect it and must be optimized
     - Varies for each workload
     - Transaction intensive apps are particularly sensitive to latency, keeping the queue lengths low and number of IOPS available high
     - Throughput intensive apps are less sensitive, queues can be long(er)
     

### Use cases

 1. Enterprise applications - Oracle, SAP, Microsoft Exchange
 1. Lift and shift
   - AWS Application Migration Service (AWS MGN)
   - AWS Server Migration Service (AWS SMS)
 1. Relational databases
 1. NoSQL databases
 1. Big Data analytics engines (Hadoop, Spark)
 1. File systems and media workflows
 1. Business continuity - Snapshots, AWS Backup
 1. Disaster recovery (for on-premises or cloud) - CloudEndure

## (Primary) Storage types

### Block storage

In computing, a **block**, sometimes called a **physical record**, is a sequence of bytes or bits, having a maximum length - **block size**. It is a raw storage in which a hardware storage device is presented as either volume or disk to be formatted and attached to OS to use. 

Storage devices can be HDDs, SSDs or newer types such as Non-Volatile Memory Express (NVMe). It can also be deployed on storage area network (SAN).

#### Architecture 

 - Block storage (either physically or logically attached to compute system)
 - Operating system (formats or makes block storage ready for use)
 - Compute system
 - (optionally) Application manages the storage

#### Management

 - **Block size**
   - is determined while formatting
   - its flexibility is fundamental differentiator
 - **Managing metadata**
   - data about data
   - helps to manage resource (permissions, type, creation/update time...)
   - tracks blocks which were used to store the data
 - **Read/write activity**
   - controlling who can read/write/delete data (can reference external sources like LDAP to do so)
   - determining how clients or applications are notified when another client is accessing the data
   - when and when the data is stored to blocks, how its cached 
   - managing read request, order, caching...
 - **Locking**
   - managing data integrity while being modified or deleted
   - file or block level locking

#### Volumes

A block storage can be a single device, or multiple devices presented as one (RAID or SAN for example) to compute system. A block storage device is referred to as **disk or drive**.

A **volume** is a logical storage construct that can be created out of one (or part of one) disk or multiple. 

#### Fast performance

Since block storage does not need to read entire files or objects (it can read just the required blocks) it is faster than other types of storage. 

##### Latency

Time between making a request (to a storage system) and receiving the response. Low latency (delay) is primary benefit of block storage. From <1ms to low two digit. 

Depends on how the storage is connected to compute. Local onsite storage is the fastest due to lack of network traversal. Onsite SAN storage is fast but has some network latency. 

Block storage has no additional network protocol overhead. Filesystem is slower. Object storage is slowest due to HTTP protocol overhead. 

##### IOPS

Input/output operations per second. Is used to measure random (very small unrelated information) I/O read-write activities.

Block storage can often obtain higher IOPS than other types. SSDs are best suited for high IOPS due to lack of seek times. 

##### Throughput

Performance associated with reading large sequential data files. Measured in MB/s (megabytes per second). HDDs are best suited for high throughput. 


### File storage

Is built on top of block storage and is created by an OS that manages underlying reading and writing of data on block storage devices. The name comes from storing data in files, typically in a directory hierarchy. 

The two most common file storage protocols are SMB (Server Message Block) and NFS (Network File System). 

### Object storage

Is also build on top of block storage. The name object storage comes from storing data as binary objects. Unlike file storage it does not differentiate between types of data it stores, its type is rather stored as metadata. 







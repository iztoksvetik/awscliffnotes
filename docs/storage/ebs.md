# Amazon Elastic Block Storage

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

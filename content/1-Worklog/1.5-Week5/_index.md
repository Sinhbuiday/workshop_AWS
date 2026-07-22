---
title: "Week 5 Worklog"
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

- Amazon Simple Storage Service ( S3 ) - Access Point - Storage Class.
- S3 Static Website & CORS - Control Access - Object Key & Performance - Glacier.
- Snow Family - Storage Gateway - Backup.

### Tasks to be carried out this week:

| Day | Task                                                  | Start Date | Completion Date | Reference Material                                                                            |
| --- | ----------------------------------------------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------- |
| 1   | - Amazon Simple Storage Service (S3)                  | 1/06/2026 | 2/06/2026      |
| 2   | - Amazon Simple Storage Service (S3) - Access Point   | 2/06/2026 | 3/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 3   | - Amazon Simple Storage Service (S3) - Storage Class  | 3/06/2026 | 4/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 4   | - S3 Static Website & CORS                            | 4/06/2026 | 5/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 5   | - Control Access - Object Key & Performance - Glacier | 5/06/2026 | 6/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |
| 6   | - Snow Family - Storage Gateway - Backup              | 6/06/2026 | 7/06/2026      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3> |

### Week 5 Achievements:

- Amazon Simple Storage Service (S3):

  - Amazon S3 is a place to store static data, meaning data that has been created and does not change afterward.
  - S3 is suitable for data types that are written once and read many times (WORM - Write Once Read Many).
  - Amazon S3 has no limit on the total amount of data stored.
  - Each object cannot be larger than **5 TB**.
  - By default, data in Amazon S3 is replicated across 3 Availability Zones (AZs) within a single Region.
  - Amazon S3 can (**trigger events**), allowing you to activate actions when an event occurs, such as uploading or deleting an object from a specific storage bucket.
  - Amazon S3 is designed for 99.99999999% **durability** and 99.99% **availability**.
  - Amazon S3 supports **multipart upload** for uploading large objects to a bucket.
  - The program creates S3 buckets to store objects in Amazon S3.
    - s3.amazonaws.com => s3.amazonaws.com/capture.mp4

- Amazon Simple Storage Service (S3) - Access Point: Amazon S3 Access Point is a feature that allows you to create unique connection points (unique hostnames) for applications, users, or groups.

  - The program can configure permissions when creating each access point.

- Amazon Simple Storage Service (S3) - Storage Class:

  - Amazon S3 divides storage into multiple storage classes to help optimize costs.
  - The storage classes of S3:
    - **S3 Standard**: Frequently accessed data.
    - **S3 Standard IA**: Infrequently accessed data.
    - **S3 Intelligent Tiering**: Automatically moves objects between storage classes based on access patterns over time.
    - **S3 One Zone IA**: Re-creatable data stored in a single zone, infrequently accessed but requiring quick access.
    - **Amazon Glacier / Deep Archive**: Cold storage for data that is rarely accessed.

- Object Life Cycle Management (configurable data retention period):

  - Move data within Amazon S3. A table defines the cases for moving data within an S3 bucket when stored data meets the configured time (days) criteria.

- Amazon Simple Storage Service (S3) - Static Website & CORS:

  - Amazon S3 has a feature that allows hosting static websites (HTML, media, etc.), making it suitable for **Single Page Applications (SPAs)**. A SPA is a web application that updates the current web page with new data from the web server using JavaScript or its frameworks such as AngularJS or ReactJS, instead of the browser’s default method of loading an entirely new page.
  - Amazon S3 supports **CORS**. **CORS** is a mechanism that allows resources (fonts, JavaScript, etc.) of a web page to be requested from a domain different from the domain of that page. CORS stands for **Cross-Origin Resource Sharing**.

- Amazon Simple Storage Service (S3) - Control Access:
  - Amazon S3 has two mechanisms for controlling access to buckets:
    - **S3 Access Control List (ACL)** is a basic access control mechanism. However, if you are already using S3 ACL and see no need for changes, S3 ACLs are attached to S3 buckets and objects. It is not always clear when AWS will use this mechanism to grant access through specific permission types.
    - **S3 Bucket Policy and IAM Policy** define object-level permissions by specifying conditions applied to the objects in the Resource section of the policy. This configuration applies to objects within the bucket. Unlike S3 ACLs, it also allows you to adjust conditions at the bucket level, giving you more flexibility in managing access.
- Amazon Simple Storage Service (S3) - Object Key & Performance:

  - Each object in S3 is flat (not hierarchical) and is assigned an object key. Example: **/image/sample.jpg, sample.jpg**.
  - Internally, S3 is divided into partitions, which are further split depending on the request load and the large number of S3 object keys (dots are used to help manage objects within a partition).
  - S3 stores a key map (the key map helps route requests to partitions based on the hash of the object key prefix).
  - To optimize S3 performance, you can use random prefixes **(e.g., /fscd/img/sample.jpg instead of /img/sample.jpg)**.
  - The goal of this approach is to distribute objects across as many partitions as possible, since S3 performance depends on the number of partitions.

- Amazon Simple Storage Service (S3) - Glacier:

  - Amazon S3 Glacier is a low-cost storage option suitable for data that does not need to be accessed frequently and is intended for long-term storage. If you need fast or frequent access, you should choose Amazon S3 instead.
  - When storing data in Amazon S3 Glacier, you cannot access it directly; you must retrieve the data back to an S3 bucket first.
  - There are three retrieval options with different access times:
    - **Expedited Retrieval**: Completed within 1–5 minutes.
    - **Standard Retrieval**: Completed within 3–5 hours.
    - **Bulk Retrieval**: Completed within 5–12 hours.

- Snow Family:

  - Snowball:
    - This service supports migrating data from an on-premises environment to AWS at a scale of up to Petabytes (PB). Each Snowball can hold up to 80 Terabytes (TB).
    - The Snowball device will be shipped to the AWS region you select to store data in S3 or Glacier.
    - The program uses the Snowball Client on the local machine to collect data and then transfer it.
  - Snowball Edge:
    - This service supports migrating data from an on-premises environment to AWS at a scale of up to Petabytes (PB). Each Snowball Edge can hold up to 100 Terabytes (TB).
    - The Snowball Edge device will be shipped to the AWS region you select to store data in Glacier or S3.
    - The program uses the Snowball Client on the local machine to collect data and then transfer it.
    - Snowball Edge is a device used to back up data from resources across the entire system when importing into the device.
  - Snowmobile:
    - This service supports migrating data from an on-premises environment to AWS at a scale of up to 100 Petabytes (PB). Each Snowmobile can hold up to 100 PB.
    - The Snowmobile will be transported to the AWS region you select to store data in services such as S3 or Glacier.

- Storage Gateway:

  - AWS Storage Gateway is a hybrid storage solution that combines AWS storage capacity with on-premises storage.
  - It is designed to scale seamlessly with cloud storage services to help store large amounts of data for long-term retention.
  - AWS Storage Gateway supports three main storage methods: file, volume, and tape:
    - **File Gateway** allows you to store and retrieve objects in Amazon S3 using the NFS or SMB file protocol. Data written through this file gateway can be accessed directly in S3.
    - **Volume Gateway** provides storage using the iSCSI protocol. Data in the volumes is stored in Amazon S3. To access iSCSI volumes in AWS, you can create EBS snapshots (optionally through AWS Backup) to generate EBS volumes.
    - **Tape Gateway** enables data backup using the VTL (iSCSI) protocol, virtual tape drives, and virtual tapes. Virtual tape data is stored in Amazon S3 or Glacier.

- Backup:
  - AWS Backup is a service that manages backup tasks. We can configure and schedule backups (backup schedule), set retention periods (backup retention), and monitor backup activities for AWS resources, including:
    - Amazon EBS
    - Amazon EC2
    - Amazon RDS databases
    - Amazon DynamoDB databases
    - Amazon EFS
    - AWS Storage Gateway volumes

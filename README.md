# AWS-Cloud-Solutions-Architect
My AWS Cloud Solutions Architect study notes

## Services

**Primary ones to focus on:  VPC, EC2, S3, EBS, ECS, LAMBDA, API, AutoScaling**

**EC2**  
Timeout isses => security groups configuration?  
Security groups can reference other security groups instead of ip ranges  
Public IP (obvious)  
* elastic IP (same as public IP, but consistent IP between stop and starts)
Private IP (obvious)  
   
* EC2 launch modes  
on demand  
reserved  
spot instances  
dedicated  
   
* Instance types:  
R = Ram  
C = CPU  
M = Medium  
I = IO  
G = GPU  
T2/T3 = Burst (or budget)  

* Amazon Machine Image (AMI)  
Create AMI to pre-install software on for EC2 => faster boot/deployment of instances  
AMI can be copeid across regions and accounts.  (default within region only)  
  
* Instance placement groups  
Cluster - close together, high performance (faster network), less redundancy  
Spread - spread among multi-AZ's, more redundancy  

**CLB / ALB / NLB**  
* CLB - may still have questions regarding security groups and stickiness  
  
* ALB  
Support routing based on hostname (users.example.com & payments.example.com)  
Support routing based on path (example.com/users & example.com/payments)  


Support redirets (HTTP to HTTPS as example)  
Support dynamic host port mapping with ECS  

* NLB (Layer 4) gets a static IP per AZ  
Public facing: must attach Elastic IP - can help whitelist by clients  
Private facing: will get random private IP base on free ones at time of creation  
Has cross zone balancing  
Has SSL termination (Jan 2019)  
  
*Restrict access to the EC2 instance by limiting security group source from the load balancer:*
![alt text](https://github.com/rlam13/AWS-Cloud-Solutions-Architect/blob/master/screenshots/load_balancer_security_groups.png)

* SSL Certificates on load balancer  
The load balancer uses a X.509 certificate (SSL/TLS server certificate)  
Manage certificates using AWS Certificate Manager (ACM)  
Alternatively create/upload own certificates
  
HTTPS listener:  
Must specify default certificate  
Optional - add list of certs to support multiple domains
Clients can use Server Name Indication (SNI) to specify the hostname they reach  
Able to specify security policy to support older versions of SSL/TLS (for legacy clients)  

  
**ASG**  
ASG Default Termination Policy:  
1. Find AZ with most instances
2. If multiple instances in AZ, delete the one with oldest launch configuration  
  
* Scaling cooldown  
Ensures that ASG doesn't launch/terminate instances before previous scaling activity  
In addition to default cooldown -> create cooldowns that apply to specific scaling policy  
Default cooldown period is 300 seconds, can reduce down to reduce scale-in policy  
If multiple scale out/in during an hours, modifty ASG cool-down timer and CloudWatch alarm period that triggers scale in  

**EBS**  
Local storage, can only be attached to one instance at a time (like a USB stick)
Locked at the AZ level  
* if you need to attach to another AZ, then backup (snapshot) and recreate in another AZ  
EBS backups use IO, don't run while application is handling lots of traffic  
Root EBS volumes get terminated by default if EC2 instance is terminated (can be disabled, if needed)  
Increase EBS volume size if disk I/O is high

**EFS**  
Network drive (file system) essentially
Can mount across lots of instances  
EFS share website files
EBS gp2, optimize cost

**Instance Store**
The volume on the instance.  Ephemeral (higher performance, but not portable/flexible like EBS/EFS)
Custom AMI for faster deploy  
  
**RDS**  
Relational DB  
Backups are enabled automatically  
  
DB Snapshots  
* manually triggered by user  
* reetention of backup as long as you need  
  
Read replicas can only SELECT (read-only), can't INSERT/DELETE/UPDATE  
Supports Transparent Data Encryption (TDE) for DB enryption  
* Oracle or SQL server DB instance only  
* TDE can be used on top of KMS - may affect performance  
  
IAM Authentication OR userid/password can be used  
Works for MySQL, PostgreSQL  
Lifespan of authentication token is 15 minutes (short-lived)  
Tokens are generated by AWS credentials   
SSL must be used when connecting to DB  
Easy to use EC2 Instance roles to connect to the RDS database  
  
**Aurora**  
Proprietary from AWS  - Postgres and MySQL are supported  
Cloud optimzed 5x improvement over MySQL on RDS, and 3x of Postgres on RDS  
Grows in increments of 10GB, up to 64TB  
Up to 15 replicas.  (MySQL can have 5), faster replication <10ms replica lag  
Failover in Aurora is instantaneous - HA native  
Costs ~20% more - but more efficient  
Auto fail-over  
Backup & Recovery  
Isolation & security  
Industry Compliance  
Push-button scaling  
Automated patching w/ zero downtime  
Advanced monitoring  
Routine maintenance  
Backtrack: restore data at any point with using backups  
No SSH (managed service)  
No need to choose instance size / helpful with unknown workload / DB scales automatically based on CPU & connections  
Can migrate from Aurora Cluster to Aurora Serverless and vice versa  
  
Aurora Serverless usage measured in Aurora Capacity Units (ACU), billed in 5 minute increment of ACU  
  
IAM Authentication for MySQL and PostgreSQL  
Aurora Global DB span multipl regions and enable DR  
* one primary region  
* one DR region  
* DR region can be used for lower latency reads  
* < 1 second replica lag on average  
If not use Global DB, can create cross-region read replics  
* FAQ recommends Global DB  
  
**Elasticache**  
Manaaged Redis or Memcached  
Very high performance, low latency  
Write scaling using sharding  
Read scaling useing read replicas  
Multi AZ with failover capability  
AWS managed. (OS maintenance/patching,optimizations,setup,config,monitoring, failure recovery and backups)  
SASL authentication (no IAM)  

**Redis**  
RedisAUTH (username/password) (no IAM)  
SSL in-flight encryption must be enabled and used  
  
**Route 53**  
Most common records to know:  
* A: URL to IPv4
* AAAA: URL to IPv6
* CNAME: URL to URL
* Alias: URL to AWS resource
    
TTL (Time to Live)  
High TTL (24hr)  
* Less traffic on DNS
* Possibly outdated records  
  
Low TTL (60s)
* more traffic on DNS  
* records are outdated for less time
* easy to change records
  
TTL is mandatory for each DNS record  
  
* AWS Resources (load balancers, cloudfront, etc) expose AWS URL:  
lbl-us-west-1.elb.amazonaws.com but you want it to appear as mycoolapp.cooldomain.com  
  
* CNAME:  
Points URL to another URL (mycoolapp.cooldomain.com >> somethingelse.com)  
Works only for NON ROOT DOMAIN ( mycoolapp.cooldomain.com)
  
* Alias:
Points a URL to AWS resource (mycoolapp.cooldomain.com >> somethingelse.amazonaws.com)
Works for ROOT DOMAIN and NON ROOT DOMAIN.  (aka cooldomain.com)
  
* Simple Routing Policy  
Maps a domain to one URL  
Used to redirect single resource  
Cannot attach health checks to Simple Routing Policy  
If multiple values are returned, random one chosen by client  
  
* Weighted Routing Policy  
Route traffic to multiple resources in proportions you specify
Helpful to test percentage of traffic on a new app version, if needed  
Can split traffic between two regions  
Can be associated with health checks  
  
* Latency Routing Policy
Redirect to the server that has the least latency  
Latency determined by user to designated AWS Region
  - Germany could be directed to the US, if that's the lower latency  

* Health Checks  
Have X health checks failed - unhealthy (default 3)  
After X health checks passed - healthy (default 3)  
Default health check interval: 30s (can set to 10s - costs more)  
Approximately 15 health checkers will check the endpoint health  
Can use HTTP/TCP and HTTPS health checks (no SSL verification)  
Possibility of integrating health check with CloudWatch 
  
Health checks can be linked to Route53 DNS queries  
  
* Failover Routing Policy  
For active/passive failover  
  
* Geo Location Routing Policy  
Routing based on user location (different from latency based)  
  
* Multivalue Answer Routing Policy  
Deploy when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random 
  
* Route 53 can be a Registar  
Domain registar is an organization that manages reservation of Internet domain names

* Third Party Registrar with Route 53  
Route 53 can be used if domain is bought from third party website
  - Create hosted zone in Route 53
  - Update NS records on third party website to use Route 53 name servers  
  
**S3**  
Objects(files) have a key.  The key is the full path:
<example_bucket_name>/example.txt  
<example_bucket_name>/sample_folder/another_folder/example.txt  
No actual directories - the UI will make it look like directories  
Object values are the content of the body:
  + max size 5TB
  + if upload is larger than 5GB, must use multi-part upload  
Metatdata (list of text key / value pairs - system or user metadata)  
Tags (unicode key/value pair - up to ten) - useful for security / lifecycle
Versioning can be enabled at the bucket level
  + same key overwrite will increment the version. IE: 1, 2, 3 etc....  
  + best practice to version buckets  
  
Files not versioned prior to enabling versioning will have version "null"  
  
* Encryption for Objects  
  + SSE-S3: AWS handles and manages keys  
    + Object is encrypted server side
    + AWS-256 encryption type
    + Must set header: "x-amz-server-side-encryption":"AES256"
  + SSE-KMS: AWS Key Management Service to manage encryption keys  
    + KMS Advantages: user control + audit trail
    + Object is encrypted server side
    + Must set header: "x-amz-server-side-encryption":"aws:kms"
  + SSE-C: self manage encryption keys  
    + Server-side encryption using data keys fully managed by the customer side
    + S3 doesn't store the encryption key
    + HTTPS must be used
    + Encryption key must provided in HTTP headers, for every HTTP request made
  + Client side encryption is an option  
    + Client library such as Amazon S3 Encryption Client
    + Client must encrypt data before sending to S3
    + Client must decrypt data after retrieving from S3
    + Encryption cycle fully managed by customer  
  
* Security
  + User based - IAM policies 
    + Set which API calls are permitted by specific user via IAM console
  + Resource based  
    + Bucket Policies - bucket wide rules from the S3 console - allows cross account
    + Object Access Control List (ACL) - finer granularity
    + Bucket Access Control List (ACL) - not as common  
    
* Bucket Policies
  + JSON based 
    + Resources buckets and objects
    + Actions: Set of API to Allow or Deny
    + Effect: Allow / Deny
    + Principal: The account or user to apply the policy to
  + Use S3 bucket for policy to:
    + Grant public access to the bucket
    + Force objects to be encrypted at upload
    + Grant access to another account (cross account)  
  
* Other
  + Networking:
    + Supports VPC Endpoints (for instance in VPC without www internet)
  + Logging and Audit
    + S3 access logs can be stored in other S3 bucket
    + API calls can be logged in AWS CloudTrail
  + User Security:
    + MFA can be required in versioned buckets to delete objects
    + Signed URL's: URL's that are valid only for a limited time (IE: premium video service for logged in users)
    
* Websites
  + S3 can host static websites 
  + The website URL would be:
    + <bucket_name>.s3-website-<AWS-region>.amazonaws.com
              OR
    + <bucket_name>.s3-website.<AWS-region>.amazonaws.com
  + If 403 error, confirm bucket policy allows public read
   
* Cross Origin Resource Sharing (CORS)
  + limits the number of webiste that request your files in S3 (saves money)
  
* Consistency Model
  + Read after write consistency for PUTS of new objects
    + as soon as object is written we can retrieve it (PUT 200 -> GET 200)
    + except if GET was done before (GET 404 -> PUT 200 -> GET 404) -eventually consistent
  + Eventual consistency for DELETES and PUTS of existing objects
    + If read object after updating, may get the older version (PUT 200 -> PUT 200 -> GET 200)
    + If delete object, may be able to retrieve it for a short time after (DELETE 200 -> GET 200)
    
* MFA Delete
    + Require MFA to permanently delete object version
    + Require MFA to suspend versioning on the bucket
    + MFA not required for enabling version
    + MFA not required for listing deleted versions
    + Only the bucket owner (root account) can enable/disable MFA delete
    + MFA delete currently enabled using the CLI

* Default Encryption vs Bucket Policies
    + old way: use bucket policy and refuse HTTP command without proper headers
    + new way: default encryption in S3 (bucket policies are evaulated before "default encryption"
  
* Access logs
    + optional: log all access to S3 buckets
    + optional: use Athena to analyze directly from a bucket
  
* Cross Region Replication
    + must enable versioing (source & destination)
    + buckets must be in different AWS regions
    + can be in different accounts
    + copying is asynchronous
    + must give proper IAM permissions to S3
    + use cases: compliance, lower latency access, replication across accounts
    
* Presigned URLs



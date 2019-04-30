# AWS-Cloud-Solutions-Architect
My AWS Cloud Solutions Architect study notes

## Services

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

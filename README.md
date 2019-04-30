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

  
**ASG**  
ASG Default Termination Policy:  
1. Find AZ with most instances
2. If multiple instances in AZ, delete the one with oldest launch configuration  
  

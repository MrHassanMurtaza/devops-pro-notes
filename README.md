# AWS Certified DevOps - Professional Study notes

These are my study notes for the AWS DevOps - Professional certification. I intend to pass the DevOps Professional exam in May 2019. And, I'll be writing my notes on a topic I study during preparation.

Table of Contents
=================

* [A/B Testing and Blue/Green Deployments](#A/B-Testing-and-Blue/Green-Deployments)

# A/B-Testing-and-Blue/Green-Deployments
- Blue/Green deployment (Blue is current running version, Green env running different version of application) is the technique we use for releasing application by shifting traffic between two identical environments running different versions of the application. Advantage: Avoid downtime (near zero downtime), Convinent in rollback.  
- Once Green environment is tested, we shift our production to green environment. Following Patterns can be used depending on use case: update DNS routing, swaping ASG behind ELB, Docker B/G deployments and updating ASG Launch configuration.
- In A/B Testing, we run two different version of application in two different environments and get it tested by users. User feedback will determine the between them. 

### → Updating DNS Routing
- Use this pattern if you have the DNS name or IP of your environment.
- Use Route53 all at once or weighted routing policies.
- Example would be having EC2 instances or having ECS container behind ELB (since ELB provides us DNS name, so we are good).

### → SWAP ASG behind ELB
- You'll use this pattern when you either can't or don't want to use Route53. 
- ELB will be responsible for routing the traffic. 
- You'll need a second auto scaling group. Overall you'll have one Blue ASG and one Green ASG. Blue is our production right now.
- We can just edit Load Balancer and attach Green ASG to our load balancer. 
- We can control the amount of traffic by altering number of instances in blue and green ASGs e.g. if Green ASG is scaled up to the size we want, we can decommission the Blue ASG by setting the instances size to 0.
- Traffic shift is quicker but we won't have a fine grain control over traffic weightage. 

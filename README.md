# AWS Certified DevOps - Professional Study notes

These are my study notes for the AWS DevOps - Professional certification. I intend to pass the DevOps Professional exam in May 2019. And, I'll be writing my notes on a topic I study during preparation.

Table of Contents
=================

* [A/B Testing and Blue/Green Deployments](#A/B-Testing-and-Blue/Green-Deployments)
* [Introduction to Elastic Beanstalk](#Introduction-to-Elastic-Beanstalk)

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

### → Update ASG Launch Configurations
- Again the case where you can't or don't want to use DNS for B/G deployments.
- We'll update launch configurations to do B/G deploymenets.
- We'll have on blue launch configuration and one green launch configuration. 
- Then we have to change our launch configuration of production ASG to Green from Blue.
- Old instances with old launch configurations will stay the same. What you need to do is to double the size of instances so that you provision new instances with new launch configurations. 
- After launching new instances we can change our ASG to previous size of instances. This way, our old instances will get terminated depending on our ASG policy. 

### → A/B Testing 
- Comparing two versions of application to see which one performs better.
- Example would be, you hosting two applications on S3 and serving using Route53 weighted policies. Then you can see conversion rate to determine which one is performing better. 

# Introduction-to-Elastic-Beanstalk
- Elastic BeanStalk reduces management complexity without restricting choice or control. You just need to upload application and elastic beanstalk will take care of capacity provision, load balancing, scaling and monitoring etc. 
- Developers friendly. 		 	
- Supported runtime includes java, .Net, PHP, NodeJS, Python, Ruby and Go. Supported webservers are Apache, IIS, Nginx, Passenger Puma, Tomcat and Docker.
- Used when you want to spend minimal time learning/configuring infrastructure. For quick prototyping and testing. 
- You wouldn't want to use it when you need complete control over resources. Complicated if you have a lot of dependencies i.e. configurations (ops work is the best fit). May not fit for existing applications.
- Elastic beanstalk components: 1. Application - the collection of components such as environments, versions and configurations. 2. Application Verson: part of an app. Can have multiple versions
- Environment - Version that is deployed with AWS resources. 
- Environment Configuration - Setting and parameters that defines our environment. 
- Configuration template - Used to create repeatable environments configurations.

### → Deployment Strategies for Elastic Beanstalk
- All at once: Update all instances at the same time with an in-place update. 
Pros - Fastest Method, Required no DNS change.
Cons - Can cause downtime. 
on Deployment Failure - Re deploy a working version.
- Rolling: 
Pros - Prevent downtime because of having control over number of instances. Uses Health Checks when re-attaching to ELB and Requires no DNS changes.   
Cons - On deployment failure, we'll have to manually terminate the instances and let elastic beanstalk deploy last successful version of the application. 
- Rolling with Additional Batch: Updates one batch of instances at a time. Starting with new batch of instances first. 
Pros - Same pros as Rolling deployments with out capacity reduction e.g. number of instances as a time. 
Cons - Takes more time than rolling updates. On deployment failure, re-deploy with another rolling additional batch deployment. Hybrid. 
- Immutable: If you want brand new instances, with out replacing batch by batch for some reasons, Immutable strategy is the way to go. It replaces existing instances with new instances by creating a temporary autoscaling group and it test one new application and configuration. Then, add it to orignal auto scaling group by terminating old auto scaling group. 
Pros - Prevents downtime. Use new resources.  In place updates with clean rollback as we are not doing anything to original instances without testing.
Cons - Doubles the number of instances for short period of time. On deployment failure, it terminates temporary ASG and re deploy.
- Blue/Green: Replaces all the resources including ELB, ASG and instances. 
Pros - Prevents downtime, uses new instances instead of in place updates and test updates on a seperate environment we call Green.
Cons - Requires DNS Change - doubles the number of instances. On deployment failures, swap urls. 

Apply to updating our environment or rolling back to the previous version. 

### → Docker Deployments with Elastic Beanstalk
- Allow us a way to package application and all of its dependencies in a virtual container. 
- It gives us consistency across environments and platforms. Especially when youu're migrating your application from lets say on premise to cloud. 
- Two Configurations of Docker with Elastic Beanstalk: Single Container: Used to deploy a docker image and source code inside a single container (Dockerfile is required & Dockerrun.aws.json is optional). Multi Container: Use to deploy multiple containers per instance. ECS is used to deploy cluster in EB environment. E.g. Nginx in one container and Application in another container (both Dockerfile & Dockerrun.aws.json is required).

### → Elastic Beanstalk Environment Configurations
- Ways to do Elastic Beanstalk Configurations: 1. Configuration Files (in .ebextensions), Saved Configurations, Direct Changes). 
- Configuration Files are added to the root of our application bundle in .ebextensions folder. 
- Saved Configurations are stored in S3. These are Yml or JSON files and can be applied to new or existing environment. 
- Anatomy of Configuration Files: 1. option_settings: values for configuration settings. Can configure eb environment, AWS resources in our environment and software on instances. 2. resources: lets you customize the resources. 3. other sections: Commands, Container commands, users etc. 
- Presidence: Direct changes have the highest precidence. Saved Configurations have the next highest. Configuration files has the lowest precidence. 
### → Elastic Beanstalk with CloudFormation
- CloudFormation: Controlled way to model and provision resources. Infrastructure as a code. 
- Elastic Beanstalk: Easily deploy and scale application without worrying about underlying resources.
- Use CloudFormation for infrastructure management and elastic beanstalk for configurations.


# Prompts used in CloudFormation Assignment. 
Main context for Claude: I gave it the assignment and updated the context throughout to represent the current stage of the project.  

### 1A Prompts & Tutorials

- Tutorial - AWS Project: How to Create a Static Website for Free Using Amazon S3, Step by Step  - https://www.youtube.com/watch?v=H83xYoC3z4A
- Tutorial - Create a Static Website on Amazon S3: The Complete Tutorial - https://www.youtube.com/watch?v=fxU7QHyxgLw

- Use managed cache policies - https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-managed-cache-policies.html
- What is Amazon S3 Object Access? - https://tutorialsdojo.com/using-origin-access-control-to-secure-s3-object-access-to-cloudfront/

1. I need a test template for CloudFormation as a YAML file that creates one private S3 bucket for a static website. Do not hard-code the bucket name. Set all four Block Public Access settings to true. Add comments explaining each choice.
2. The free tier of AWS is limited in these ways: IAM Identity Center is unavailable, console sign in for IAM is unavailable, MFA is unavailable, region can not be changed. Can I still proceed with these limitations? How can I use the CLI to input commands? 
3. Why should I create a separate IAM user for daily work?
4. What is the least privilege principle, and what are the trade-offs of giving my IAM user AdministratorAccess for this lab?
5. How to run and verify the commands from the CLI and where do I find documentation? 
6. I need a basic index.html file containing my student name (Sean Doyle) and my college number (L00205287). Include description "CloudFormation Lab, task 1A: static website hosted from a private S3 bucket." I will do my own comments.  
7. How to upload html file to bucket through aws CLI?
8. I am being recommended to enable bucket versioning by Amazon, how could I enable this through AWS CLI commands? 
9. Tell me about CloudFront and its relevance to this project. We have got the index.html stored in a bucket. What is CloudFront going to do for us? What are the reasons to use CloudFront vs other options?
10. What do I need to add to template.yaml to use CloudFront? 
11. With the yaml updated, what is the next step? 
12. The yaml was successfully validated, how do I proceed?
13. How do I empty an S3 Bucket and delete all the bucket versions through AWS CLI? Can you explain those commands?
14. Where do I find the CloudFront domain name?  
15. What can you tell me about CloudFormation Outputs section? 
16. After deploying the test template, how do I check through the CLI that all four Block Public Access settings are enabled on the bucket?
17. Where to add versioning to the template? 
18. Am I compeletely safe to delete the stack and redeploy for testing? 



### 1B Prompts & Tutorials

- Tutorial: Amazon Web Services (AWS) 3-Tier Architecture: A Real Client Project | Part 1 https://www.youtube.com/watch?v=aNee9Z3fSdw
- Tutorial: AWS Three-Tier Architecture Deployment Step-by-Step | VPC, Subnets, ALB, Auto Scaling, RDS - IE - https://www.youtube.com/watch?v=yeyr6OBMfcE

1. What do we need to for a basic three tier architecture .yaml file for CloudFormations? 
2. Can you generate me a basic three tier architecture template.yaml file with a web tier, application tier, database tier? Just basics, no comments. 
3. Generate the web tier portion of the .yaml?
4. Why do the load balancer and the RDS (Relational Database Service) subnet group require subnets in at least two AZs (Availability Zones)?
5. How should the route tables differ for the public, application and database subnets?
6. Can I use HTTPS if I do not have a domain name?
7. how do I avoid hard-coding AZ names?
8. Should security groups allow traffic by IP range or by reference to another security group?
9. Add a private RDS PostgreSQL instance to the template.
10. Where to store database password without it being a security risk?
11. Which AMI are the cheapest? 
12. What does the application tier need to connect to the database and read the secret?
13. How do I give EC2 instances permission to read only one Secrets Manager secret? Explain the difference between the trust policy and the permissions policy.#
14. How to get AMI ID and store it as a parameter? I want to avoid hard-coding where possible.
15. I need a launch template that installs a bare bones python web app.  The app needs to query PostgreSQL and read the secret.
16. Tell me about auto scaling groups
17. What is DependsOn for the NAT route and listener? 
18. What is ELB and health checks?
19. Add an auto scaler group for two subnets to the template with health checks.
20. What do we need for a web tier? 
21. Add the internet facing load balancer first.  
22. Can we stop the health check querying the database?


# SeanDoyle-L00205287-AWS-CloudFormation-Assignment

Repository of Sean Doyle - Student Number: L00205287 Assignment: AWS Cloud Formation Module: IAC for DevOps Pipelines Tutor: Ruth Lennon.


### Power Tools
- PyCharm IDE for code analysis and Version Control
- Claude LLM 

## Part 1A

### Stage 1 

- Constraint: New AWS accounts are region locked by default, so we must use eu-north-1.
- Constraint: Tried to enable IAM Identity Center to enable Multifactor Authentication, but it is not allowed for free account.  

Upon creating my AWS account, I first had to secure the root account with multifactor authentication, then I would make a separate account for daily work. 
- An IAM users CLI credentials stay valid until you delete them, whereas an Identity Centers credentials expire after a few hours, giving better security.  A leaked permanent key lets anyone use your account until you notice or revoke it

Created IAM user sean-admin with AdministratorAccess Policy.  This way we can avoid permission errors, although a narrower policy of least privilege would be more secure typically
- AWS recommends aws login and CloudShell as alternatives to access keys but they both depend on console sign in which we don't have access to on the free tier, so both would have to run as the root user.  I judged this to be a bigger security risk as the root users permissions could not be restricted, and has no MFA, whereas the access keys can be deleted and the IAM user permission's can be reduced.  
- The key will be kept far outside the repository and deleted after the assignment.  
 
Testing Template 
- Created trial template to test AWS CLI, credentials, and that everything works together when creating and deleting resources, any later failure would come with an addition and not in the setup. 
- A public bucket would have still worked and gave no warning it was publicly readable, so we needed to check.

Claude helped with the commands and finding CloudFormation documentation on the CLI commands: https://docs.aws.amazon.com/cli/latest/reference/cloudformation/ 
I had Claude explain the test template and I recorded the prompts in [Prompts.md](Prompts.md)


### Stage 2 - Upload index.html to the Bucket ###

In stage 2 we will put the website page in the private S3 bucket from stage 1. The template only creates the bucket, so if we need to update the page, we can do that separately without needing to update the stack.   

Now we need to create a page to put in our bucket. I will have Claude create a basic html file containing my student number, name, and a description for now. 

I will upload to the bucket through AWS CLI instead of using the AWS S3 browser dashboard to avoid using the root user.  
Upon Amazons recommendation, I am enabling bucket versioning through the CLI which would allow me to recover any files that get overwritten in a bucket.

- Amazon is giving me a few different options for storage, but I will just go with the Standard storage class.  Standard storage class being designed for frequent access, which we will do through CloudFront, and there is no fees or minimum storage.  


### Stage 3 - Host the Website with CloudFront ###
CloudFront is AWS's Content Delivery Network, which is used to help make the bucket reachable to the public.  CloudFront provides HTTPS
as a security measure, caches content close to visitors which helps performance, and caches our user responses to absorb traffic, which helps scalability.  


I kept the bucket private, so the only way to read the site's files is through CloudFront (where HTTPS and caching apply), and a wrong upload can never become publicly readable.
If we want to use CloudFront to keep our bucket access private, then we need an OAC (Origin Access Control).  This lets CloudFront sign requests to S3, which verifies that they came from our distribution. Only our distribution can read the bucket, which bumps up security.

Claude helped me to generate the additional code for the CloudFront resources to be represented in the template.yaml file.

Upon deployment, I checked to make sure the bucket still had public access blocked.  


### Stage 4 - Clean State Verification ###
It is time to make sure that the template alone can rebuild the whole site but the stack has version control we would need to empty all the versioned buckets as well before we delete the stack.  

I initially enabled the version control through the AWS CLI, it has now been added it into the template.yaml for future deployments.

Claude helped me with the AWS CLI commands for deleting all the object versions and delete markers before we deleted the stack.  

The template alone rebuilt the whole site from a torn-down state. The page header shows my L-number and name, and I confirmed this by opening the CloudFront address in a browser.


### Stage 5 - Outputs Section ###

These three values are generated when the stack is created, so they are different after every rebuild. After each redeploy I would have to look up the bucket name, distribution ID and CloudFront domain name each time, which would have to be done manually each time, which is tedious and time-consuming.
To remove these extra steps, I added an `Outputs` section to `static-website/template.yaml` to help facilitate the redeployment process.

`Outputs` is its own section, separate from `Resources` in template.yaml. When the stack deploys, CloudFormation will store the values, where I can then read all three values
easily through a single query command: `aws cloudformation describe-stacks --stack-name static-website-stack --query "Stacks[0].Outputs" --output table`.  This CLI command could also be scripted to speed up the process further. 

I added three outputs:

- `BucketName` (used to upload `index.html`)
- `DistributionId` (used for CloudFront CLI commands)
- `SiteUrl` (the public HTTPS address of the website)


I validated the template with `aws cloudformation validate-template`, redeployed with the same `aws cloudformation deploy` command as before,
and read the three values with the query command above. 

To avoid ongoing charges, I emptied the bucket (including all object versions and delete markers) and then deleted the stack after confirming everything could be rebuilt successfully.

Beyond the basics, I added bucket versioning to the template and an Outputs section to remove the manual lookups after each rebuild.


## Part 1B

Purpose: builds a three-tier architecture in one CloudFormation stack.

- Web tier: internet-facing Application Load Balancer (ALB) in public subnets
- Application tier: EC2 (Elastic Compute Cloud) instances in private subnets, managed by an Auto Scaling Group (ASG), running a small Python web app
- Database tier: private, encrypted RDS (Relational Database Service) PostgreSQL instance

Traffic flow: internet -> ALB (port 80) -> application instances (port 80) -> database (port 5432).
Each hop is restricted by a security group that only accepts traffic from the previous tier.

### Stage 1 - Network and Security Groups

- For some resilience later on in the web and application tiers, we have two AZs (Availability Zones), some other aspects of 1B also require a minimum of two AZs regardless, such as the load balancer and RDS (Relational Database Service) subnet groups.
- The database subnets won't require a default route as the database will only talk to the application tier and won't need internet.  
- Network Address Translation (NAT) is needed so that the app instances can download packages from the internet as well as reach the secrets manager later on. The availability trade off is that this is a single point of failure but the cost will be lower for a single NAT Gateway instead of one per AZ.  
- HTTP only because no domain for a certificate
- A security group was made for each tier, but instead of allowing the traffic from an IP range, we instead allow the traffic between the chained security groups. These rules will stay the same even if the IP changes.  

### Stage 2 - Database Tier

- Added a private, encrypted RDS (Relational Database Service) PostgreSQL instance.
- The password is generated by RDS and stored in Secrets Manager, so it is not in the template or repository.
- Single AZ (`MultiAZ: false`) to keep cost down. Trade-off: if that AZ fails, the database is unavailable.
- A production setup would keep a final snapshot and enable deletion protection. Here we have set `DeletionPolicy: Delete`, `UpdateReplacePolicy: Delete` and `DeletionProtection: false` so the stack can be deleted entirely without further charges.
- The instance class (`db.t4g.micro`) is set as a parameter to lower cost and can be changed without editing the resource through CLI with `--parameter-overrides DbInstanceClass=db.t4g.small`.

### Stage 3 - Application Tier

- Application tier runs code that talks to the database.  
- Private application subnets with no public address.  
- Only load balancer security group can reach them on port 80
- IAM (Identity and Access Management) controls which AWS API calls an identity can make.  
- Application needs 1 API call to read database password from secrets manager. Password is not in template. Fetched at runtime.
- The Amazon Machine Image (AMI) ID gets resolved through a Systems Manager (SSM) parameter when deployed, instead of being hardcoded. This is done because the machine image IDs can be different depending on the region (and can go out of date).
- Added autoscaling for both application subnets. Health checks (`HealthCheckType: ELB`) happen at intervals and if an instance doesn't respond after a set number of times, it is replaced with a `healthy` instance.  Being conscious of cost, the desired capacity is 1 and the max is two.  It does not scale on traffic.

### Stage 4 - Web Tier

- Added an internet-facing ALB (Application Load Balancer) in the two public subnets, using `LoadBalancerSecurityGroup`. Visitors connect to the load balancer and application instances stay in private subnets with no public address.

- Added a target group with a health check on `/health` and an HTTP listener on port 80 that forwards to it. `/health` does not query the database, so a database problem does not mark an instance unhealthy.
  - Any normal path, such as `/`, returns the page with the database result.
  - The path `/health` returns just the text `ok`.
- Verified: the `SiteUrl` page shows `L00205287 Sean Doyle` and `Database connection successful`, and the target reports `healthy`.
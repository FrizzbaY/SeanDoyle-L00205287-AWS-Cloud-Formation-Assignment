# SeanDoyle-L00205287-AWS-Cloud-Foundations-Assignment
Repository of Sean Doyle - Student Number: L00205287
Assignment: AWS Cloud Foundations
Module: IAC for DevOps Pipelines
Tutor: Ruth Lennon.

### Stage 1 ###

- Constraint: New AWS accounts are region locked by default, so we must use eu-north-1.
- Constraint: Tried to enable IAM Identity Center to enable Multifactor Authentication, but it is not allowed for free account.  

Upon creating my AWS account, I first had to secure the root account with multifactor authentication, then I would make a separate account for daily work. 
- Constraint: AWS free tier does not allow access to IAM Identity Center
- An IAM users CLI credentials stay valid until you delete them, whereas an Identity Centers credentials expire after a few hours, giving better security.  A leaked permanent key lets anyone
- use your account until you notice or revoke it

Created IAM user sean-admin with AdministratorAccess Policy.  This way we can avoid permission errors, although a narrower policy of least privilege would be more secure typically
- Constraint: Identity Center, IAM console access and root MFA were unavailable on a new Free Plan account, so I used an IAM user with an access key, which is a security trade-off. 
- AWS recommends aws login and CloudShell as alternatives to access keys but they both depend on console sign in which we don't have access to on the free tier, so both would have to run as the roo user.  I judged this to be a bigger security risk as the root users permissions could not be restricted, and has no MFA, whereas the access keys can be deleted and the IAM user permission's can be reduced.  
- The key will be kept far outside the repository and deleted after the assignment. IAM user permissions can also be adjusted for the sean-admin account.  
- I cannot stress enough that the access key is a compromise and not a benefit
 

Testing Template 
- Created YAML template to test AWS CLI, credentials, to test that everything works together when creating and deleting resources, any later failure would come with an addition and not in the setup. 
- I deployed the stack for testing from my CLI, but because the IAM user has no console access with the free plan, I had to check for successful deployment and what it contained, as well as whether the bucket was private.  
- A public bucket would have still worked and gave no warning it was publicly readable, so we needed to check.



I ran `aws cloudformation deploy` to create the stack from static-website/template.yaml, then `aws cloudformation describe-stack-resources`
to confirm the bucket was created, and checked the bucket access block was private with `aws s3api get-public-access-block --bucket name`, 
and deleted the stack with `aws cloudformation describe-stacks --stack-name` once everything was confirmed to be working as intended.

Claude helped with the commands and finding CloudFormation documentation on the CLI commands: https://docs.aws.amazon.com/cli/latest/reference/cloudformation/ 
I had Claude explain the test template and I recorded the prompts in [Prompts.md](Prompts.md)


### Stage 2 - Upload index.html to the Bucket ###

In stage 2 we will put the website page in the private S3 bucket from stage 1. The template only creates the bucket, so if we need to update the page, we can do that separately without needing to update the stack.   

Now we need to create a page to put in our bucket. I don't do a lot of front-end development, so I will have Claude create a basic html file containing my student number, name, and a description for now. 

- I'm going to change the colour of the text and the background colour because it is quite difficult to look at.

I will upload to the bucket through AWS CLI instead of using the AWS S3 browser dashboard to avoid using the root user.  
Upon Amazons recommendation, I am enabling bucket versioning through the CLI with `aws s3api get-bucket-versioning --bucket static-website-stack-sitebucket-xxxxxxxxxxx` which would allow me to recover any files that get overwritten in a bucket.  A precautionary measure that I think makes perfect sense.     

- Amazon is giving me a few different options for storage but I will just go with the Standard storage class.  Standard storage class being designed for frequent access, which we will do through CloudFront, and there is no fees or minimum storage.  


### Stage 3 - Host the Website with CloudFront ###
CloudFront is AWS's Content Delivery Network, which is used to help make the bucket reachable to the public.  CloudFront provides HTTPS
as a security measure, caches content close to visitors which helps performance, and caches our user requests to absorb traffic, which helps scalability.  

If we want to use CloudFront to keep our bucket access private, then we need an OAC (Origin Access Control).  This lets CloudFront sign requests to S3,
which verifies that they came from our website. Only our distribution can read which bumps up security.

Claude helped me to generate the additional code for the CloudFront resources to be represented in the template.yaml file.  The next step was to validate the template with `aws cloudformation validate-template` 
which runs checks on file prior to creating any resources.  Simple checks such as correct syntax and structure. 

With a successful validation of the code, it was time to deploy through the CLI with `aws cloudformation deploy --template-file static-website/template.yaml --stack-name static-website-stack`.  

When we tried to deploy, we got an error saying access denied with 403 error.  403 error indicating that the server understood the request but refused us. 
S3 was not allowing CloudFront to read the S3 objects, so a site bucket policy needed to be made the yaml file, allowing read only access to the 
CloudFront ARN (Amazon Resource Name).  

Once again, the template was validated before continuing, and I ran the deployment command again.  


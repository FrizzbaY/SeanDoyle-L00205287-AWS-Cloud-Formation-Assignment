# SeanDoyle-L00205287-AWS-Cloud-Formation-Assignment
Repository of Sean Doyle - Student Number: L00205287
Assignment: AWS Cloud Formation
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
and deleted the stack with `aws cloudformation delete-stack --stack-name` once everything was confirmed to be working as intended.

Claude helped with the commands and finding CloudFormation documentation on the CLI commands: https://docs.aws.amazon.com/cli/latest/reference/cloudformation/ 
I had Claude explain the test template and I recorded the prompts in [Prompts.md](Prompts.md)


### Stage 2 - Upload index.html to the Bucket ###

In stage 2 we will put the website page in the private S3 bucket from stage 1. The template only creates the bucket, so if we need to update the page, we can do that separately without needing to update the stack.   

Now we need to create a page to put in our bucket. I don't do a lot of front-end development, so I will have Claude create a basic html file containing my student number, name, and a description for now. 

- I'm going to change the colour of the text and the background colour because it is quite difficult to look at.

I will upload to the bucket through AWS CLI instead of using the AWS S3 browser dashboard to avoid using the root user.  
Upon Amazons recommendation, I am enabling bucket versioning through the CLI with `aws s3api put-bucket-versioning --bucket static-website-stack-sitebucket-xxxxxxxxxxx --versioning-configuration Status=Enabled` which would allow me to recover any files that get overwritten in a bucket.  A precautionary measure that I think makes perfect sense.     

- Amazon is giving me a few different options for storage, but I will just go with the Standard storage class.  Standard storage class being designed for frequent access, which we will do through CloudFront, and there is no fees or minimum storage.  


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

Deployment went smoothly, everything up and running, checked to make sure the bucket still had public access blocked.  
Using the Client URL command `curl --head https:xxxxxxx.cloudfront.net/` to fetch the HTTP response headers, I noticed that the X-Cache, which tells us
how CloudFront handled the request, had a `Miss from cloudfront`, meaning cloudfront did not issue us the cached copy.  Upon running
the curl command again this resolved itself with a `Hit from cloudfront`, which makes sense as the S3 file would not have been cached until after the initial request.  


### Stage 4 - Clean State Verification ###
It is time to make sure that the template alone can rebuild the whole site.  The first thing needed to do is
to delete stack.  The stack does have version control though, so we need to empty all the versioned buckets as well before we delete the stack.  

I initially enabled the version control through the AWS CLI, so I have to add it into the template.yaml to keep that settings enabled when redeploying in the future.
I revalidate the template once again before continuing. 

Claude helped me with the AWS CLI commands for deleting all the object versions and delete markers before we deleted the stack.  
To delete the stack we used `aws cloudformation delete-stack --stack-name static-website-stack` and I checked the stack was deleted with `aws cloudformation describe-stacks --stack-name static-website-stack`.

I now test deployment again and see if the changes to the template.yaml are working. - `aws cloudformation deploy --template-file static-website/template.yaml --stack-name static-website-stack`
The deployment was successful, but the AWS Management Console is not available to check the resources, because the IAM user has no console access. Instead, the check was done through the AWS CLI. 
The `describe-stack-resources` command lists each resource in the stack, and the ResourceStatus of CREATE_COMPLETE for each of the four resources confirms they were created.

Our redeployed bucket is currently empty, and the values for the bucket name, distribution domain and distribution ID 
will be different this time around. 

We can see the BucketName on the S3 Buckets Dashboard or using `describe-stack-resources`, which we will need to upload the index.html to our Bucket.
I run `aws s3 cp static-website/index.html s3://static-website-stack-sitebucket-new-template-name/index.html` and check the upload finished with `aws s3 ls s3://static-website-stack-sitebucket-new-template-name/`. 

In that same output for `describe-stack-resources` we see the CloudFront distribution ID, and we can run `aws cloudfront get-distribution --id XXXXXXX` to get our CloudFront DomainName output. 
`$ curl --head https://xxxxxxxxx.cloudfront.net/` returns Hit from Cloud on second request.  
`get-public-access-block` still shows our Block Public Access is still all true.  

The template alone rebuilt the whole site from a torn-down state. Looking up the new values by hand after every redeploy was slow, so the next improvement is an Outputs section in the template.
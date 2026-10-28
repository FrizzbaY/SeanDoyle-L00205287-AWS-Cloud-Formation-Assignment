# SeanDoyle-L00205287-AWS-Cloud-Foundations-Assignment
Repository of Sean Doyle - Student Number: L00205287
Assignment: AWS Cloud Foundations
Module: IAC for DevOps Pipelines
Tutor: Ruth Lennon.




- Constraint: New AWS accounts are region locked by default, so we must use eu-north-1.
- Constraint: Tried to enable IAM Identity Center to enable Multifactor Authentication, but it is not allowed for free account.  

Upon creating my AWS account, I first had to secure the root account with multifactor authentication, then I would make a separate account for daily work. 
- Constraint: AWS free tier does not allow access to IAM Identity Center
- An IAM users CLI credentials stay valid until you delete them, whereas an Identity Centers credentials expire after a few hours, giving better security.  A leaked permanent key lets anyone
- use your account until you notice or revoke it

Created IAM user sean-admin with AdministratorAccess Policy.  This way we can avoid permission errors, although a narrower policy of least privilege would be more secure typically
- Constraint: Identity Center, IAM console access and root MFA were unavailable on a new Free Plan account, so I used an IAM user with an access key, which is a security trade-off. 
- AWS recommends aws login and CloudShell as alternatives to access keys but they both depend on console sign in which we don't have acess to on the free tier, so both would have to run as the roo user.  I judged this to be a bigger security risk as the root users permissions could not be restricted, and has no MFA, whereas the access keys can be deleted and and the IAM user permission's can be reduced.  
- The key will be kept far outside the repository and deleted after the assignment. IAM user permissions can also be adjusted for the sean-admin account.  
- I cannot stress enough that the access key is a compromise and not a benefitgit s
 



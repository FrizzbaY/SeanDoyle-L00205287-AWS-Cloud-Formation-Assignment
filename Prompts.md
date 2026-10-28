Prompts used in CloudFormation Assignment. 

I used a Claude Project with context instruction for each chat, and Deepseek, which is free, to save some token usage where I could for more simple inquiries.

Main context for Claude: I gave it the assignment and updated the context throughout to represent the current stage of the project.  

1. I need a test template for CloudFormation as a YAML file that creates one private S3 bucket
for a static website. Do not hard-code the bucket name. Set all four Block
Public Access settings to true. Add comments explaining each choice.

2. The free tier of AWS is limited in these ways: IAM Identity Center is unavailable, console sign in for IAM is unavailable, MFA is unavailable, region can not be changed. Can I still proceed with these limitations? How can I use the CLI to input commands? 
3. Asked how to run and verify the commands from the CLI, asked for documentation

4. I need a basic index.html file containing my student name (Sean Doyle) and my college number (L00205287). Include description "CloudFormation Lab, task 1A: static website hosted from a private S3 bucket." I will do my own comments.  
5. How to upload html file to bucket through aws CLI? 

6. I am being recommended to enable bucket versioning by Amazon, how could I enable this through AWS CLI commands? 
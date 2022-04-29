# Ansible-vault-secret-manage-using-AWS-secret-manager

In this article am securing ansible vault encrypt key using AWS secret manager service. Normal we need to remeber the vault encrypt password or we should save it somewhere in server or your local system, and there is no privacy or security to that key, so we save the key in AWS secret manager.

## Description

We will be using ansible vault to encrypt the sensitive field of the ansible vault key and store it in AWS secret manager. This can be achieved by the following:-


* Ansible encrypt the secret varribale file

* While building the property file contact AWS secrets manager for vault key.

* Retrive the key from AWS secret manager.

* Build the property file and place the file (with decrypted value) in the target host.

## Pre-Requestes (Packages Installation)

* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed on your server.

* jq installed on your server

* An ansible file with encryption


## Procedure

## Step 1. Encrypt ansible yml file

~~~sh
ansible-vault encrypt encryption.yml
~~~

## Step 2. Create IAM role 

Craete IAM role with secret manager read and write policy added in AWS IAM service. Once this role is created, attach it to manager EC2 instance. You can either use custom code for policy or existing policy. You can use this offical documention for [IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

> To create an IAM role using the IAM console

  1. Open the IAM console at https://console.aws.amazon.com/iam/.
  2. In the navigation pane, choose Roles, Create role.
  3. On the Select role type page, choose EC2 and the EC2 use case. Choose Next: Permissions.
  4. On the Attach permissions policy page, select an AWS managed policy that grants your instances access to the resources that they need.
  5. On the Review page, enter a name for the role and choose Create role.

> To attach an IAM role to an instance

   1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
   2. In the navigation pane, choose Instances.
   3. Select the instance, choose Actions, Security, Modify IAM role.
   4. Select the IAM role to attach to your instance, and choose Save.

## Step 3. Create secret in secret manager

To create a secret, you can use the console to enter the secret details.

> To create a secret

   1. Open the Secrets Manager console at https://console.aws.amazon.com/secretsmanager/.
   2. Under Secrets, choose Store a new secret.
   3. On the Store a new secret page, do the following:
      a. For Secret type, choose Other type of secret.
      b. For Key/value pairs, in the first field, enter Password. In the second field, enter a password. This will be encrypted when you save the secret.
      c. For Encryption key, keep DefaultEncryptionKey to use the AWS managed key for Secrets Manager. There is no cost for using this key.
      d. Choose Next.
   4. On the Secret name and description page, for Secret name, enter TutorialSecret, and then at the bottom of the page, choose Next.
   5. On the Secret rotation page, keep Disable automatic rotation, and then at the bottom of the page, choose Next.
   6. On the Review page, review the secret details, and then choose Store.

Secrets Manager console returns to the list of secrets in your account and the new secret is now in the list.

> To retrieve a secret

   1. Open the Secrets Manager console at https://console.aws.amazon.com/secretsmanager/.
   2. On the Secrets list page, choose TutorialSecret.
   3. On the Secrets details page, in the Secret value section, choose Retrieve secret value.

You can view your secret as a key value pair or on the Plaintext tab as JSON.

You can also create secret using CLI, for creating secret from CLI follow these steps:

  1. Create a JSON file (encrypt.json) with encryption key
     {
    "ansible_vault_password": "value used for encrypting ansible yml file"
     }
  2. Run AWS cli command for generating secret
     ~~~sh
     aws secretsmanager create-secret --name ansible/vaultpassword  --description "Secret" --secret-string file://ecncrypt.json --region ap-south-1
     ~~~
  3. Result:
       ```
       {
        "VersionId": "d04f008c-48f1-4398-8555-3e8f87b78d0a", 
        "Name": "ansible/vaultpassword", 
        "ARN": "arn:aws:secretsmanager:ap-south-1:320070737204:secret:ansible/vaultpassword-38cA1C"
       }
       ```
Don't forget to remove the json file we created after generating the secret.

## Step 4. Retrive the secret created

Here, we use bash script for retiving the secret from secret manager so that we can use this script file along with the ansible to decrypt the yml file.

> To retrive secret from secret manager
~~~sh
vim secret.sh
~~~
Add this:
~~~
#! /bin/bash

secret=${aws secretsmanager get-secret-value --secret-id ansible/vaultpassword --region ap-south-1 --query SecretString --output text |  cut -d: -f2 | tr -d \"}
echo ${secret}
~~~

Pass execution permission to the script file (secret.sh)
~~~
chmod +x secret.sh
~~~

## Step 5. Run the ansible yml file

~~~sh
ansible-playbook ansible.yml --vault-password-file ./secret.sh
~~~

## Conclusion

In this article, we created ansible vault to encrypt the sensitive field of the application properites and store the ansible vault key in AWS secret manager and retrive the ecryption key using bash script. Please contact me if you have any questions in this section. Thank you!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>

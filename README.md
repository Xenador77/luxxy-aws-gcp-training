# luxxy-aws-gcp-training
This project demonstrates:
- How to create AWS and GCP users and add policies to it through User Group Policies
- How to deploy infrastructure as a code using Terraform
- Leverage kubernetes and Docker to host our application in a containerized fashion 
- How to migrate on-premises application files and database to a multicloud environment

## Solution Architecture
![alt text](https://thecloudbootcamp.com/wp-content/uploads/2022/08/EN-PPT-MISSAO-1-2-3-ICP2_FINAL.png)

## Requisites

AWS and GCP account, AVOID using root user account

## Part 1

Deploy project architecture on AWS and GCP using Terraform

### Amazon Web Services (AWS)

#### IAM 

Create User Group called S3 and add AmazonS3FullAccess policy to it

Create User called terraform and associate it to S3 user group

On Security credentials tab, create Access Keys and download it as a .CSV file

Change the name of the file to accessKeys.csv

### Google Cloud Platform (GCP)

Download project files from https://tcb-public-events.s3.amazonaws.com/icp/mission1.zip

Connect to Cloud Shell and manually upload .zip and .csv files

Check if files have finished uploading using ```ls -la```

Run the following code to prepare the files
```
mkdir mission1_en
mv mission1.zip mission1_en
cd mission1_en
unzip mission1.zip
mv ~/accessKeys.csv mission1/en
cd mission1/en
chmod +x *.sh
```

The following command will configure and set our environment on AWS and GCP
```
./aws_set_credentials.sh accessKeys.csv
gcloud config set project <your-project-id>
```

Run the following sh script to set the project on Google Cloud Shell
```
./gcp_set_project.sh
```
Enable  Kubernetes, Container Registry and Cloud SQL APIs
```
gcloud services enable containerregistry.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable sqladmin.googleapis.com
```

Open Google Cloud Editor and edit the tcb_aws_storage.tf file, line 4 to a unique S3 bucket name

Run these commands to use Terraform to allocate the necessary infrastructure resources
```
cd ~/mission1_en/mission1/en/terraform/
terraform init
terraform plan
terraform apply
```
Type "yes" to confirm when prompted

Wait until Cloud SQL finishes creating the DB

#### Configure SQL Network 

Open Cloud SQL service and the newly created SQL instance

On the left panel under Primary Instance, click Connections

Go to Network tab, then Instance IP assignment and enable Private IP

Under Associated Network select default then Set Up Connection

Enable Service Network API and set to auto allocate IP range, then click Continue

Afterwards, on Connections -> Authorized Networks -> Add Network, set the following information:
```
Name: Public Access - Test
Network: 0.0.0.0/0
```
Do not set public access to a PRODUCTION database, this is just a test case so it will not be a problem since in the end we will delete every resource created

## Part 2
Configure our application to use a MultiCloud environment leveraging Docker and Kubernetes

### Amazon Web Services (AWS)

#### IAM 

Create a User named uxxy-covid-testing-system-en-app1 and associate it to the S3 user group

On the Security credentials tab, create Access Keys and download it as a .CSV file

### Google Cloud Platform (GCP)
Open Cloud SQL Service and open the previously created instance

Create a new user named app and set the password to welcome123456

Connect to Google Cloud Shell

Download project files programmatically using
```
cd ~
mkdir mission2_en
cd mission2_en
wget https://tcb-public-events.s3.amazonaws.com/icp/mission2.zip
unzip mission2.zip
```

Connect to MySql, check the instance public IP address and adjust the following code
```
mysql --host=<public_ip_cloudsql> --port=3306 -u app -p
```

Create a new table
```
use dbcovidtesting;
source ~/mission2/en/db/create_table.sql
show tables;
exit;
```

Enable Cloud Build API
```
gcloud services enable cloudbuild.googleapis.com
```

Build a Docker file and send it over to the Google Container Registry
```
cd ~/mission2_en/mission2/en/app
gcloud builds submit --tag gcr.io/<PROJECT_ID>/luxxy-covid-testing-system-app-en
```

Go to Cloud Editor and edit the Kubernetes Deployment File (luxxy-covid-testing-system.yaml) 
>Line 33, add your GCP Project ID

>Line 42, your AWS Bucket ID

>Line 44 and Line 46, add user access credentials

>Line 48, add Cloud SQL Private IP

Open Google Kubernetes Engine service

Deploy the application to the Cluster
```
cd ~/mission2_en/mission2/en/kubernetes
kubectl apply -f luxxy-covid-testing-system.yaml
```

Check if the application is up and running through its Public IP: GKE -> Services & Ingress -> Endpoint

## Part 3
Migrate On-Premises application files and Database to AWS and GCP

### Google Cloud Platform (GCP)

#### Migrating MySql DB

Connect to Cloud Shell
Download DB dump files 
```
cd ~
mkdir mission3_en
cd mission3_en
wget https://tcb-public-events.s3.amazonaws.com/icp/mission3.zip
unzip mission3.zip
```
Connect to MySql using welcome123456 for the password
```
mysql --host=<public_ip_address> --port=3306 -u app -p
```

Import dump files to Cloud SQL
```
use dbcovidtesting;
source ~/mission3_en/mission3/en/db/db_dump.sql
```

Verify that the table contains all data
```
select * from records;
exit;
```

### Amazon Web Services (AWS)
#### Migrate PDF files

Connect to AWS Cloud Shell
Download PDF files programmatically
```
mkdir mission3_en
cd mission3_en
wget https://tcb-public-events.s3.amazonaws.com/icp/mission3.zip
unzip mission3.zip
```
Sync the files with the S3 Bucket
```
cd mission3/en/pdf_files
aws s3 sync . s3://<S3-BUCKET-ID>
```
See if the application updates its data over "View results" tab

DONE! We have simulated a migration from an on-premises architecture to a multicloud one! 

**Don't forget to delete every resource created on AWS and GCP!**

## Acknowledgements

Many thanks to @Jean Rodrigues

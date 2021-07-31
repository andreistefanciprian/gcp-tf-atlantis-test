# Description

This sample code build resources in GCP cloud.

## Run terraform

```bash
# define vars
export GCP_PROJECT='changeMeGcpProject'
export GCP_ZONE=change-me2-c
export GCP_REGION=change-me2

# create GCP Service Account to be used by Atlantis
gcloud iam service-accounts create terraform \
--description="Used by Terraform" \
--display-name=terraform

# grant service account IAM GCP roles
gcloud projects add-iam-policy-binding $GCP_PROJECT \
--member=serviceAccount:terraform@${GCP_PROJECT}.iam.gserviceaccount.com \
--role=roles/compute.admin

gcloud projects add-iam-policy-binding $GCP_PROJECT \
--member=serviceAccount:terraform@${GCP_PROJECT}.iam.gserviceaccount.com \
--role=roles/storage.admin

# verify Service Account was created
gcloud iam service-accounts list
gcloud iam service-accounts describe terraform@${GCP_PROJECT}.iam.gserviceaccount.com

# create service accoutn keys
gcloud iam service-accounts keys create ${GCP_PROJECT}-sa.json \
--iam-account=terraform@${GCP_PROJECT}.iam.gserviceaccount.com

# move sa file to safe location
mv ${GCP_PROJECT}-sa.json ~/${GCP_PROJECT}-sa.json

# have terraform service account key file as environment variable
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/${GCP_PROJECT}-sa.json"

# create bucket
gsutil mb -b on -l $GCP_REGION gs://$GCP_PROJECT-tfstate

# check bucket was created
gsutil ls

# run terraform
terraform init -var-file=environments/dev/terraform.tfvars  environments/dev
terraform plan -var-file=environments/dev/terraform.tfvars  environments/dev
terraform apply -input=false -auto-approve -var-file=environments/dev/terraform.tfvars  environments/dev
terraform destroy -input=false -auto-approve -var-file=environments/dev/terraform.tfvars  environments/dev
```
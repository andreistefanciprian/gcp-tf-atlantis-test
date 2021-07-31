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

# create global bucket
gsutil mb gs://$GCP_PROJECT-tfstate

# check bucket was created
gsutil ls

# build terraform infra
cd environments/dev
terraform init \
-backend-config="bucket=$GCP_PROJECT-tfstate" \
-backend-config="prefix=terraform/dev"
terraform plan
terraform apply -input=false -auto-approve

# destroy terraform infra
terraform destroy -input=false -auto-approve
```
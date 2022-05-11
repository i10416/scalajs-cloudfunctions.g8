
# $name$

[![Deploy]($githubRepo$/actions/workflows/deploy.yaml/badge.svg)]($githubRepo$/actions/workflows/deploy.yaml)


$description$


## prerequisites
- node
- sbt
- gcloud command
- gsutil command

## Debug

```sh
sbt fullOptJS
```


```sh
npm install -g @google-cloud/functions-framework
```

```sh
functions-framework --target=$functionName$
```

## Deploy


### Setup GCP


```sh
gcloud auth application-default login
```


```sh
gcloud services enable iamcredentials.googleapis.com --project "$GCPProjectID$"
```


Create Service Account for Terraform



```sh
gcloud iam service-accounts create terraform --display-name "ci/cd terraform executor"
```

```sh
gcloud projects add-iam-policy-binding $GCPProjectID$ \
  --member=serviceAccount:terraform@$GCPProjectID$.iam.gserviceaccount.com \\
  --role=roles/editor
```

```sh
gcloud projects add-iam-policy-binding $GCPProjectID$ \
  --member=serviceAccount:terraform@$GCPProjectID$.iam.gserviceaccount.com \\
  --role=roles/cloudfunctions.admin
```


Configure Workload Identity Pool and Provider


```sh
export POOL_NAME=\$POOL_NAME
```

```sh
cloud iam workload-identity-pools create "\$POOL_NAME" \
    --project=$GCPProjectID$ --location="global"
```

```sh
export WORKLOAD_IDENTITY_POOL_ID=\$( \
    gcloud iam workload-identity-pools describe "\$POOL_NAME" \
      --project="$GCPProjectID$" --location="global" \
      --format="value(name)" \
  )
```

```sh
export GH_USER=
```

```sh
export GH_REPO=
```

```sh
gcloud iam service-accounts add-iam-policy-binding terraform@$GCPProjectID$.iam.gserviceaccount.com \
    --project="$GCPProjectID$" \
    --role="roles/iam.workloadIdentityUser" \
    --member="principalSet://iam.googleapis.com/\${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/\${GH_USER}/\${GH_REPO}"
```


### Environment Variables


#### Github Actions

| name                        | example value                                                                                  | description                |
| --------------------------- | ---------------------------------------------------------------------------------------------- | -------------------------- |
| TF_CLI_VERSION              | 1.1.19                                                                                         | terraform version          |
| TF_GCP_SERVICE_ACCOUNT      | terraform@$GCPProjectID$.iam.gserviceaccount.com                                               | terraform service account  |
| GCP_PROJECT_ID              | $GCPProjectID$                                                                                 | gcp project id             |
| GCP_REGION                  | $GCPRegion$                                                                                    | gcp region                 |
| GCP_FUNCTION_NAME           | $functionName$                                                                                 | cloud function entrypoint  |
| GCP_FUNCTION_BUCKET         | $functionBucket$                                                                               | cloud function bucket name |
| GCP_WORKLOAD_ID_PROVIDER_ID | projects/project_number/locations/global/workloadIdentityPolls/\$POOL_NAME/providers/oidc-name |                            |


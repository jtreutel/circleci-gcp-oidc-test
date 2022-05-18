# circleci-gcp-oidc-test
Example repo showing how to authenticate to GCP using a CircleCI OIDC token.


## Usage

**Basic Usage**

Add the vars in [the table below](#required-environment-vars) to a context.  Then, use the command shown in the [example config below](#sample-configyml) to authenticate to GCP.

**Using Multiple Service Accounts from the Same GCP Project**

If you need to use multiple service accounts in the same project, you can override the GCP_SERVICE_ACCOUNT_EMAIL environment variable at the job level. See the `gcp-oidc-override-sa` job in this repo's config.yml for an example.

**Using Multiple Service Accounts from Multiple GCP Projects**

The cleanest way to interact with multiple GCP projects would be to create a context to store the variables for each project and then reference them accordingly ([example here](#multi-gcp-project-configyml-snippet)).

**Reusing a Stored Credentials File in Multiple Jobs**

If you do not wish to authenticate in each job, you can store the credentials file in a [workspace](https://circleci.com/docs/2.0/workspaces/). See the `gcp-oidc-generate-and-store-creds` and `gcp-oidc-reuse-creds` jobs in this repo's config.yml for an example.  

&nbsp;&nbsp;


### Required Environment Vars

Add these to a context:

| Context var name | Example value | Notes |
|---|---|---|
|GCP_PROJECT_ID| `123456789012`	| [GCP project number](https://cloud.google.com/resource-manager/docs/creating-managing-projects#before_you_begin)
|GCP_WIP_ID|	`myworkloadpoolid`	| [Workload identity pool ID](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers#pools)|
|GCP_WIP_PROVIDER_ID| `myproviderid` | [Workload identity pool provider name](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers#manage-providers)|
|GCP_SERVICE_ACCOUNT_EMAIL|	`myserviceacct@myproject.iam.gserviceaccount.com`	| [User-managed Service Accounts](https://cloud.google.com/iam/docs/service-accounts#user-managed)|

&nbsp;&nbsp;

### Sample config.yml

```yaml
version: "2.1"

orbs:
  gcp-cli: circleci/gcp-cli@2.4.1

commands:
  gcp-oidc-generate-creds:
    description: "Authenticate with GCP using a CircleCI OIDC token."
    parameters:
      project_id: 
        type: env_var_name
        default: GCP_PROJECT_ID
      workload_identity_pool_id: 
        type: env_var_name
        default: GCP_WIP_ID
      workload_identity_pool_provider_id: 
        type: env_var_name
        default: GCP_WIP_PROVIDER_ID
      service_account_email: 
        type: env_var_name
        default: GCP_SERVICE_ACCOUNT_EMAIL
      gcp_creds_file_path: 
        type: string
        default: /home/circleci/gcp_creds.json
      oidc_token_file_path: 
        type: string
        default: /home/circleci/oidc_token.json
    steps:
      - run:
          command: |
            # Store OIDC token in temp file
            echo $CIRCLE_OIDC_TOKEN > << parameters.oidc_token_file_path >>
            # Create a credential configuration for the generated OIDC ID Token
            gcloud iam workload-identity-pools create-cred-config \
                "projects/${<< parameters.project_id >>}/locations/global/workloadIdentityPools/${<< parameters.workload_identity_pool_id >>}/providers/${<< parameters.workload_identity_pool_provider_id >>}"\
                --output-file="<< parameters.gcp_creds_file_path >>" \
                --service-account="${<< parameters.service_account_email >>}" \
                --credential-source-file=<< parameters.oidc_token_file_path >>

  gcp-oidc-auth:
    description: "Authenticate with GCP using a GCP credentials file."
    parameters:
      gcp_creds_file_path: 
        type: string
        default: /home/circleci/gcp_creds.json
    steps:
      - run:
          command: |
            # Configure gcloud to leverage the generated credential configuration
            gcloud auth login --brief --cred-file "<< parameters.gcp_creds_file_path >>"
            # Configure ADC
            echo "export GOOGLE_APPLICATION_CREDENTIALS='<< parameters.gcp_creds_file_path >>'" | tee -a $BASH_ENV

jobs:
  gcp-oidc-defaults:
    executor: gcp-cli/default
    steps:
      - gcp-cli/install
      - gcp-oidc-generate-creds
      - gcp-oidc-auth

workflows:
  main:
    jobs: 
      - gcp-oidc-defaults:
          name: GCP OIDC Auth
          context: 
          - gcp-oidc-dev
```

&nbsp;&nbsp;

### Multi-GCP Project config.yml snippet

```yaml
#jobs, orbs, etc omitted

workflows:
  main:
    jobs: 
      - deploy-to-staging:
          context: 
          - gcp-staging-context
      - deploy-to-prod:
          context: 
          - gcp-prod-context
```
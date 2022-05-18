# circleci-gcp-oidc-test
Testing the best way to auth to GCP using OIDC tokens.


### Usage

Set these vars in a context:

| Context var name | Example value | Notes |
|---|---|---|
|GCP_PROJECT_ID| `123456789012`	| [GCP project number](https://cloud.google.com/resource-manager/docs/
|GCP_WIP_ID|	`myworkloadpoolid`	| [Workload identity pool ID](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers#pools)|
|GCP_WIP_PROVIDER_ID| `myproviderid` | [Workload identity pool provider name](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers#manage-providers)|
|GCP_SERVICE_ACCOUNT_EMAIL|	`myserviceacct@myproject.iam.gserviceaccount.com`	| [User-managed Service Accounts](https://cloud.google.com/iam/docs/service-accounts#user-managed)|

# amcaughan-personal

Aaron McAughan's projects: infrastructure, data pipelines and analytics on AWS, built and run the way a small platform team would run them.

## The projects

One data flow, end to end, each stage in its own repository:

```
data-simulator-api  ──►  data-simulator-elt  ──►  data-simulator-analytics
synthetic data API       hourly ingest to S3       dbt models + dashboard
```

| Repository | What it is | See it |
|---|---|---|
| [data-simulator-api](https://github.com/amcaughan-personal/data-simulator-api) | A public API that generates realistic synthetic data from a spec: tables or an endless time-windowed stream. FastAPI on Lambda behind CloudFront and WAF, staging then canary deploys to production | [API reference](https://amcaughan-personal.github.io/data-simulator-api/) |
| [data-simulator-elt](https://github.com/amcaughan-personal/data-simulator-elt) | Pulls the API's payments feed every hour into S3 as raw JSON, with catch-up after failures and a contract check on the response, queryable in Athena | [Design decisions](https://github.com/amcaughan-personal/data-simulator-elt/blob/main/docs/design-decisions.md) |
| [data-simulator-analytics](https://github.com/amcaughan-personal/data-simulator-analytics) | dbt models on Athena (bronze, silver, gold) over the raw data, run daily in a container Lambda, feeding a static dashboard | [Dashboard](https://amcaughan-personal.github.io/data-simulator-analytics/) |

## The Kubernetes platform

A shared EKS cluster for a small data team, run the way a platform team would run it: brought up from a GitHub workflow you approve, and taken down every night so it costs nothing while unused.

| Repository | What it is |
|---|---|
| [eks-platform](https://github.com/amcaughan-personal/eks-platform) | The cluster and what it runs. Terraform creates EKS, Karpenter and Argo CD; Argo CD installs everything else from git; Karpenter adds spot nodes as pods need them and removes them when they empty. A scoped IAM role brings clusters up and down from CI, and a teardown script checks nothing is left behind |
| [data-pipelines](https://github.com/amcaughan-personal/data-pipelines) | In progress: Airflow pipelines on that cluster, each task its own pod, pulling from the data-simulator API into S3 and Athena |


| Repository | What it is |
|---|---|
| [personal-infra](https://github.com/amcaughan-personal/personal-infra) | The AWS account baseline (CloudTrail alarms, GuardDuty, budgets and cost anomaly alerts), the IAM roles CI uses, and this organization's own settings, all as Terraform |
| [shared-workflows](https://github.com/amcaughan-personal/shared-workflows) | Reusable GitHub Actions workflows every repository calls: Terragrunt plan and apply, Python checks, security scans, Kubernetes manifest validation |
| [dev-environment](https://github.com/amcaughan-personal/dev-environment) | The dev container base image and dotfiles |
| [repo-template](https://github.com/amcaughan-personal/repo-template) | What every new repository starts from |

## How it's run

- **Everything as code.** AWS infrastructure and GitHub settings (repositories, branch rules, required checks, deployment environments) are Terraform, managed with Terragrunt.
- **Pull requests only.** The main branch is protected in every repository, and changes merge when their checks pass. Pull requests that touch infrastructure get the Terraform plan posted as a comment.
- **Infrastructure applied from CI.** Merging to main applies each project's infrastructure to dev, then to production after an approval. A daily drift check fails if AWS no longer matches main. The account baseline itself is applied by hand, so CI can never change its own permissions.
- **Least-privilege CI.** GitHub Actions signs in to AWS with short-lived OIDC credentials (no stored keys), as a separate role for each job: read-only for plans, create-and-change but never delete for applies, and every role a project creates carries a permissions boundary.
- **Deploys through CI.** Application code ships from main: staging or dev first, smoke-tested, then production.
- **Shared, pinned CI.** Common workflows live in one repository and are released as version tags. Projects pin them by commit, and Dependabot proposes updates.
- **Security checks on every pull request:** Checkov, Trivy, Gitleaks and actionlint.

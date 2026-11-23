# amcaughan-personal

Aaron McAughan's projects: infrastructure, data pipelines and analytics on AWS, built and run the way a small platform team would run them.

## What's here

- **Foundations:** the AWS account baseline and this organization's own settings, as Terraform; reusable CI workflows; a shared dev container; and the template every new repository starts from.
- **Projects:** each in its own repository, with its application code, its own infrastructure (Terraform and Terragrunt) and its own docs.

## How it's run

- **Everything as code.** AWS infrastructure and GitHub settings (repositories, branch rules, required checks, deployment environments) are Terraform. Changes are reviewed in pull requests and applied by hand after reviewing the plan.
- **Pull requests only.** The main branch is protected in every repository, and changes merge when their checks pass. In project repositories, pull requests that touch infrastructure get the Terraform plan posted as a comment.
- **Deploys through CI.** Application code ships from main with GitHub Actions, which signs in to AWS with short-lived OIDC credentials. No stored keys.
- **Shared, pinned CI.** Common workflows live in one repository and are released as version tags. Projects pin them by commit, and Dependabot proposes updates.
- **Security checks on every pull request:** Checkov, Trivy, Gitleaks and actionlint.

# .github

The organization's profile page: GitHub shows [`profile/README.md`](profile/README.md) at the top of
[github.com/amcaughan-personal](https://github.com/amcaughan-personal). Edit it with a pull request like any other change.

## Development environment

Open the repo in its dev container (VS Code: *Reopen in Container*). It's the org's base
image (Ubuntu, AWS CLI, [mise](https://mise.jdx.dev)) plus the tools listed in
`.tool-versions`; add tools there as the repo needs them. Python projects also set
`.python-version`.

## Checks

Every pull request runs the org's shared security scan: leaked secrets (gitleaks),
vulnerable dependencies (Trivy), workflow problems (actionlint) and infrastructure
misconfigurations such as in the dev container's Dockerfile (Checkov).

`pre-commit install` adds the same secret check to every local commit.

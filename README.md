# 👩‍💻 Conventions

Our Priceloop Engineering conventions for Python, Golang, Git Workflow etc.

## 📖 Contents
- [Python Styling Guide](#python-styling-guide)
- [Golang Styling Guide](#golang-styling-guide)
- [Naming Conventions](#naming-conventions)
- [Git Commit Structure](#git-commit-structure)
- [Git Branch Flow](#git-branch-flow)
- [Secrets](#secrets)
- [Language](#language)
- [License](#license)

## Python Styling Guide

Use the following pre-commit hooks:
* isort
* pylint following [Google styling guide](https://google.github.io/styleguide/pyguide.html)
* black formater with the following flags:
    * `-l 110 -S -t`
    * VSCode settings.json
```json
"python.formatting.blackArgs": ["-l", "110", "-S", "-t"]
```
* flake8
* docstrings: [Google conventions](https://google.github.io/styleguide/pyguide.html)
* Quotes: double
* mypy
* Use pre-commit with the following hooks:
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.2.3
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: debug-statements
      - id: requirements-txt-fixer
      - id: flake8

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v0.720
    hooks:
      - id: mypy
        args: [--allow-redefinition, --ignore-missing-imports]

  - repo: local
    hooks:
      - id: isort
        name: "Sort imports"
        language: system
        types: [file, python]
        entry: isort
      - id: pylint
        name: "PyLint"
        language: system
        types: [file, python]
        files: .
        exclude: test_*.py
        entry: python3 -m pylint
      - id: black
        name: "Black"
        language: system
        pass_filenames: false
        entry: black .
        args: [--safe, --quiet, "--line-length=110", "--skip-string-normalization"]
```

## Golang Styling Guide
Use pre-commit with the following hooks:
```yaml
default_stages: [commit, manual]

repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.4.0
    hooks:
        - id: trailing-whitespace
        - id: end-of-file-fixer
        - id: check-yaml
  - repo: git://github.com/dnephin/pre-commit-golang
    rev: master
    hooks:
      - id: go-fmt
      - id: go-vet
      - id: go-mod-tidy
```

### Tips & Conventions
Use [effective go](https://golang.org/doc/effective_go) guide for all your inquiries and standards. Effective go includes all the conventions specific to Go language including
* variable, package and function naming
* packaging and modules
* concurrency
* error handling
* formatting

[Illustrations](https://peter.bourgon.org/go-best-practices-2016/#top-tip-3) of good practices (may be slightly outdated)

## Naming Conventions

### Git Repositories
```
priceloop/priceloop-{{logical_layer}}-{{repo_name}}
```

e.g.

```
priceloop/priceloop-data_science-price_elasticity
priceloop/priceloop-data_engineering-data_ingress_service
priceloop/priceloop-client_backend-auth_service
```

### Git Branch Naming

Your branch name should follow the format type-scope.

Types:
* feat
* fix
* chore

### Storage buckets, i.e. s3, gcs
```
ai.priceloop.{{bucket_purpose}}.{{environment}}
```
e.g. `ai.priceloop.data.prod`

`{{environment}} = {prod, stage}`

`{{bucket_purpose}}` can potentially be broken down to

`{{layer}}.{{bucket_purpose}}`, e.g.
```
ai.priceloop.ml.metadata
ai.priceloop.frontend.assets
ai.priceloop.dataplatform.metadata
```

### Namespace
* Long: priceloop-production / priceloop-staging
* Short: priceloop-prod / priceloop-stage

## Git Commit Structure
* Be descriptive
* Use all lower-case
* Limit punctuations (no dots, no commas)
* Include one of the specified types
  * docs: Documentation only changes
  * feat: A new feature
  * fix: A bug fix
  * refactor: A code change that neither fixes a bug nor adds a feature
  * test: Adding missing tests or correcting existing tests
  * chore: updating grunt tasks etc; no production code change
* Short (under 70 characters is best)
* In general, follow the [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/#summary) guidelines

## Git Branch Flow

We follow the following PRs flow with user facing feature(s):

```
develop → { git checkout -b } → feature branch → {PR} develop → {PR} master
```

### CI/CD

We use Github Actions as the CI solution. Every repository with the codebase for user facing features must contain the following CI configuration/workflow definition files:

```bash
.github
└── workflows
    ├── develop.yaml
    ├── master.yaml
    └── pr_branches_check.yaml
```

Where
* `develop.yaml` - defines the CI pipeline to deploy to the stage env
* `master.yaml` - defines the CI pipeline to deploy to the prod env
* `pr_branches_check.yaml` - defines the check to validate if PR is compliant with the branch flow

pr_branches_check.yaml
```yaml
name: PR Branches Check

on:
  pull_request:
    types:
      - opened
      - reopened

jobs:
  develop_to_master:
    if: github.base_ref == 'master'
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR to "master" is from "develop"
        if: ${{ github.head_ref != 'develop' }}
        run: |
          echo "Wrong source branch, only develop->master PR's allowed"
          exit 1
  any_to_develop:
    if: github.base_ref == 'develop'
    runs-on: ubuntu-latest
    steps:
      - name: Check if PR is to "develop"
        if: ${{ github.head_ref == 'refs/heads/master' }}
        run: |
          echo "Wrong source branch, only "!master"->develop PR's allowed"
          exit 1
```

### GitHub Branches Protection Rules
Every Github repository containing user facing features must have the protection rules for the `master` and `develop` branches.

## Secrets

⚠️ Never put any username, password on Git, always use local environments or secrets manager to access services

* Metadata considered as secrets:
  * AWS keys (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
  * Access tokens (any oauth2 tokens which have long life time)
  * Access credentials, i.e. combination of password and user name
  * Webhook URLs
* Metadata which can be stored to git:
  * Bucket name, path to object
  * Database name
  * Resource ID, URI ← it may be a subject of change/dispute

Note: in case of doubt whether the info is a secret/internal or can be public, treat it at a higher security level, i.e. as a secret. It costs nothing to downgrade security level, but it's quite expensive to upgrade it 😃

### How to share secrets
To share between peers:
* Please use [gpg](https://gnupg.org/) to encrypt opaque text secret.
* Slack DM is acceptable, but remember to delete the secret message after it was received!

We use [AWS Secret Manager](https://aws.amazon.com/secrets-manager/) and [github secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) for secrets to be used programmatically.

## Language
* Stick to American English e.g. optimize instead of optimise or behavior instead of behaviour. In doubt check Google 😃

## License
* For private repos, no License necessary
* For public repos, we should use [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0.html)
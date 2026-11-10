# Onward Logistics order system: bootstrap

Things that must exist before the order app can be built or the infrastructure can be deployed. Deploy this first.

Infrastructure and architecture diagram: [onward-orders-infrastructure](https://github.com/effienjoki/onward-orders-infrastructure)

## What it creates

Stack `onward-dev-orders-bootstrap`, from `infrastructure/bootstrap.yaml`:

| Resource | Why it must exist first |
| --- | --- |
| ECR repository `onward-dev-orders` | The app pipeline pushes images here, and the ECS service pulls from here. Scan on push is on. A lifecycle rule keeps the 10 newest images |
| GitHub OIDC provider | Makes AWS trust tokens issued by GitHub Actions. One per account, so the template can create it or reuse an existing one (`CreateGitHubOidcProvider`) |
| App push role and policy | Assumed by the app repo's `main` branch only. Can only push to the ECR repository |
| Templates bucket | Holds the child templates for the infrastructure's nested stacks. Versioned, private, old versions expire after 30 days |
| Infrastructure deploy role and policy | Assumed by the infrastructure repo's `main` branch only. Can upload templates and ask CloudFormation to deploy the `onward-dev-ecs` stacks using the deploy role. Cannot create resources itself |

Both GitHub roles match the OIDC token's subject on immutable owner and repository IDs, so a recreated account or repository with the same name cannot assume them.

## Outputs

| Output | Used by |
| --- | --- |
| `RepositoryName`, `RepositoryUri`, `RepositoryArn` (exported) | The infrastructure root stack |
| `GitHubActionsRoleArn` | App repo variable `AWS_ROLE_ARN` |
| `InfraActionsRoleArn` | Infrastructure repo variable `AWS_INFRA_ROLE_ARN` |
| `TemplatesBucketName` | Infrastructure repo variable `TEMPLATES_BUCKET` |

## Parameters

Set in `infrastructure/bootstrap-deployment.yaml`. `GitHubOwnerId`, `AppRepoId` and `InfraRepoId` are the immutable IDs GitHub includes in its OIDC token. Find a repository's ID at `https://api.github.com/repos/<owner>/<repo>` (the first `id` field).

## Deploy

CloudFormation Git sync, stack `onward-dev-orders-bootstrap`, deployment file `infrastructure/bootstrap-deployment.yaml`. Git sync suits this repository because it has a single template.
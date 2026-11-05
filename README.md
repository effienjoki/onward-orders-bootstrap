# onward-orders-bootstrap
# Onward Logistics order system: bootstrap

Things that must exist before the order app can be built or run. Deploy this first.

Infrastructure and architecture diagram: [onward-orders-infrastructure](https://github.com/effienjoki/onward-orders-infrastructure)

## What it creates

Stack `onward-dev-orders-bootstrap`, from `infrastructure/bootstrap.yaml`:

| Resource | Why it must exist first |
| --- | --- |
| ECR repository `onward-dev-orders` | The app pipeline pushes images here, and the ECS service pulls from here. Scan on push is on. A lifecycle rule keeps the 10 newest images |
| GitHub OIDC provider | Makes AWS trust tokens issued by GitHub Actions. One per account, so the template can create it or reuse an existing one (`CreateGitHubOidcProvider`) |
| GitHub Actions role and push policy | Assumed by the app repo's `main` branch only. Can only push to this one ECR repository |

## Exports

`RepositoryName`, `RepositoryUri` and `RepositoryArn`, used by the service and delivery stacks.

The `GitHubActionsRoleArn` output goes into the app repo as the GitHub variable `AWS_ROLE_ARN`.

## Parameters

Set in `infrastructure/bootstrap-deployment.yaml`. `GitHubOwnerId` and `AppRepoId` are the immutable IDs GitHub includes in its OIDC token. To find them, print the token's `sub` claim from a workflow run.

## Deploy

CloudFormation Git sync, stack `onward-dev-orders-bootstrap`, deployment file `infrastructure/bootstrap-deployment.yaml`.
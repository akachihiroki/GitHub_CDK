# Welcome to your CDK TypeScript project

This is a blank project for CDK development with TypeScript.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

## Useful commands

* `npm run build`   compile typescript to js
* `npm run watch`   watch for changes and compile
* `npm run test`    perform the jest unit tests
* `npx cdk deploy`  deploy this stack to your default AWS account/region
* `npx cdk diff`    compare deployed stack with current state
* `npx cdk synth`   emits the synthesized CloudFormation template

## GitHub Actions CI/CD

This repository includes two workflows:

* `.github/workflows/ci.yml`
	* Trigger: pull request, push to `main`
	* Runs: `npm ci` -> `npm run build` -> `npm run test` -> `npx cdk synth`

* `.github/workflows/deploy.yml`
	* Trigger: push to `main`, manual `workflow_dispatch`
	* Uses GitHub OIDC to assume an AWS IAM Role and runs `npx cdk deploy --require-approval never`

### Required GitHub settings

Set the following in your GitHub repository settings:

* `Secrets`
	* `AWS_ROLE_ARN`: IAM role ARN that GitHub Actions can assume

* `Variables` (optional)
	* `AWS_REGION`: target region (default is `ap-northeast-1`)

For OIDC, make sure the IAM role trust policy allows your repository. Example:

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Principal": {
				"Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
			},
			"Action": "sts:AssumeRoleWithWebIdentity",
			"Condition": {
				"StringEquals": {
					"token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
				},
				"StringLike": {
					"token.actions.githubusercontent.com:sub": "repo:<OWNER>/<REPO>:*"
				}
			}
		}
	]
}
```

### One-time preparation

Before using the deploy workflow, bootstrap CDK in the target account/region once:

* `npx cdk bootstrap aws://<ACCOUNT_ID>/<REGION>`

After bootstrap and settings, merging into `main` will automatically deploy.

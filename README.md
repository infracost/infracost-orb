# Infracost CircleCI Orb

[This CircleCI Orb](https://circleci.com/developer/orbs/orb/infracost/infracost) runs [Infracost](https://infracost.io) against pull requests and automatically adds a pull request comment showing the cost estimate difference for the planned state. See [this repo](https://github.com/infracost/circleci-github-demo) for a demo of the Orb being used with GitHub, and [this repo](https://bitbucket.org/infracost/circleci-bitbucket-demo) for a demo of it being used with Bitbucket.

Since Bitbucket [does not](https://community.atlassian.com/t5/Bitbucket-questions/View-all-comments-on-a-pull-request/qaq-p/677092) show commit comments in the pull request page, the Orb posts a pull request comment if applicable; otherwise it posts a commit comment (only visible in the commit's comments page). Similarly for GitHub we post pull request comments by default but if the Orb runs before a pull request is created it will post a commit comment. If possible, we recommend users to select the "Only build pull requests" option in CircleCI's Project > Advanced settings to always get the pull request comments.

The Orb uses the latest version of Infracost by default as we regularly add support for more cloud resources. If you run into any issues, please join our [community Slack channel](https://www.infracost.io/community-chat); we'd be happy to guide you through it.

As mentioned in our [FAQ](https://infracost.io/docs/faq), no cloud credentials or secrets are sent to the Cloud Pricing API. Infracost does not make any changes to your Terraform state or cloud resources.

<img src="screenshot.png" width=557 alt="Example screenshot" />

## Table of Contents

* [Usage](#usage)
* [Parameters](#parameters)
* [Environment variables](#environment-variables)
* [Contributing](#contributing)

# Usage

1. In CircleCI, go to your Project Settings > Environment Variables, and add environment variables for `INFRACOST_API_KEY`, either `GITHUB_TOKEN` or `BITBUCKET_TOKEN`, and any other required credentials (e.g. `AWS_ACCESS_KEY_ID`).

2. Create a new file at `.circleci/config.yml` in your repo with the following content. Use the Parameters section above to decide which options work for your Terraform setup. The following example uses `path` to specify the location of the Terraform directory and `terraform_plan_flags` to specify the variables file to use when running `terraform plan`.

    ```
    version: 2.1
    orbs:
      infracost: infracost/infracost@0.8.4
    workflows:
      main:
        jobs:
          - infracost/infracost:
              path: path/to/code
              terraform_plan_flags: -var-file=my.tfvars
    ```

    If you already run Terraform commands and generate a plan JSON before using the Infracost Orb, you can use Orb `pre-steps` to `attach_workspace` so the Infracost Orb has access to the plan JSON file, e.g.:

    ```
    jobs:
      - infracost/infracost:
          pre-steps:
            - attach_workspace:
                at: /workspace/.terraform
          path: /workspace/.terraform/tfplan.json
    ```

3. Send a new pull request to change something in Terraform that costs money; a comment should be posted on the pull request. Check the CircleCI logs and [this page](https://www.infracost.io/docs/troubleshooting/) if there are issues.

## Parameters

### `path`

**Optional** Path to the Terraform directory or JSON/plan file. Either `path` or `config_file` is required.

### `terraform_plan_flags`

**Optional** Flags to pass to the 'terraform plan' command, e.g. `"-var-file=my.tfvars -var-file=other.tfvars"`. Applicable when path is a Terraform directory.

### `terraform_workspace`

**Optional** The Terraform workspace to use. Applicable when path is a Terraform directory. Only set this for multi-workspace deployments, otherwise it might result in the Terraform error "workspaces not supported".

### `usage_file`

**Optional** Path to Infracost [usage file](https://www.infracost.io/docs/features/usage_based_resources#infracost-usage-file) that specifies values for usage-based resources, see [this example file](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) for the available options.

### `config_file`

**Optional** If your repo has **multiple Terraform projects or workspaces**, define them in a [config file](https://www.infracost.io/docs/config_file/) and set this input to its path. Their results will be combined into the same diff output. Cannot be used with path, terraform_plan_flags or usage_file inputs. 

### `show_skipped`

**Optional** Show unsupported resources, at the bottom of the Infracost output (use a string value, either "true" or "false", default is "false").

### `post_condition`

**Optional** A JSON string describing the condition that triggers pull request comments, can be one of these:
- `'{"update": true}'`: we suggest you start with this option. When a commit results in a change in cost estimates vs earlier commits, the integration will create **or update** a PR comment (not commit comments). The GitHub comments UI can be used to see when/what was changed in the comment. PR followers will only be notified on the comment create (not update), and the comment will stay at the same location in the comment history. This is the default behavior for GitHub, please let us know if you'd like to see this for GitLab and BitBucket.
- `'{"has_diff": true}'`: a commit comment is put on the first commit with a Terraform change (i.e. there is a diff) and on every subsequent commit (regardless of whether or not there is a Terraform change in the particular commit). This is the current default for GitLab, BitBucket and Azure Repos (git).
- `'{"always": true}'`: a commit comment is put on every commit.
- `'{"percentage_threshold": 0}'`: absolute percentage threshold that triggers a comment. For example, set to 1 to post a comment if the cost estimate changes by more than plus or minus 1%. A commit comment is put on every commit with a Terraform change that results in a cost diff that is bigger than the threshold.

Please use [this GitHub discussion](https://github.com/infracost/infracost/discussions/1016) to tell us what you'd like to see in PR comments.

### `sync_usage_file` (experimental)

**Optional**  If set to `true` this will create or update the usage file with missing resources, either using zero values or pulling data from AWS CloudWatch. For more information see the [Infracost docs here](https://www.infracost.io/docs/usage_based_resources#1-generate-usage-file). You must also specify the `usage_file` input if this is set to `true`.

## Environment variables

This section describes the main environment variables. Other supported environment variables are described in the [this page](https://www.infracost.io/docs/integrations/environment_variables).

Terragrunt users should also read [this page](https://www.infracost.io/docs/iac_tools/terragrunt). Terraform Cloud/Enterprise users should also read [this page](https://www.infracost.io/docs/iac_tools/terraform_cloud_enterprise).

### `INFRACOST_API_KEY`

**Required** To get an API key [download Infracost](https://www.infracost.io/docs/#quick-start) and run `infracost register`.

### `GITHUB_TOKEN`

**Optional** GitHub token used to post comments (e.g. a Personal access token), needs to have `repo` scope so it can post comments.

### `BITBUCKET_TOKEN`

**Optional** Bitbucket "username:password" used to post comments (e.g. "myusername:my_app_password"), the password needs to have Read scope on "Repositories" and "Pull Requests" so it can post comments. Using a [Bitbucket App password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) is recommended.

### Cloud credentials

**Required** You do not need to set cloud credentials if you use Terraform Cloud/Enterprise's remote execution mode, instead you should follow [this page](https://www.infracost.io/docs/iac_tools/terraform_cloud_enterprise).

For all other users, the following is needed so Terraform can run `init`:
- AWS users should set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, or read [this section](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#environment-variables) of the Terraform docs for other options. If your Terraform project uses multiple AWS credentials you can configure them using the [Infracost config file](https://www.infracost.io/docs/multi_project/config_file/#examples). We have an example of [how this works with GitHub actions here](https://github.com/infracost/infracost-gh-action#multiple-aws-credentials).
- GCP users should set `GOOGLE_CREDENTIALS` or read [this section](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/provider_reference#full-reference) of the Terraform docs for other options.

### `INFRACOST_TERRAFORM_BINARY`

**Optional** Used to change the path to the `terraform` binary or version, see [this page](https://www.infracost.io/docs/integrations/environment_variables/#cicd-integrations) for the available options.

### `GIT_SSH_KEY`

**Optional** If you're using Terraform modules from private Git repositories you can set this environment variable to your private Git SSH key so Terraform can access your module.

### `SLACK_WEBHOOK_URL`

**Optional** Set this to also post the pull request comment to a [Slack Webhook](https://slack.com/intl/en-tr/help/articles/115005265063-Incoming-webhooks-for-Slack), which should post it in the corresponding Slack channel.

## Contributing

Issues and pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## Publishing

To publish Orb, you can amend the commit or push another commit with [semver:FOO] in the subject where FOO is 'major', 'minor', or 'patch'. CircleCI will automatically bump the Orb version based on the semver commit.

To indicate intention to skip promotion, include [semver:skip] in the commit subject instead.

## License

[Apache License 2.0](https://choosealicense.com/licenses/apache-2.0/)


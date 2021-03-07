# Infracost CircleCI Orb

This [CircleCI Orb](https://circleci.com/developer/orbs/orb/infracost/infracost) runs [Infracost](https://infracost.io) against pull requests whenever Terraform files change. It automatically adds a pull request comment showing the cost estimate difference for the planned state if a percentage threshold is crossed. See [this repo](https://github.com/infracost/circleci-github-demo) for a demo of the Orb being used with GitHub, and [this repo](https://bitbucket.org/infracost/circleci-bitbucket-demo) for a demo of it being used with Bitbucket.

Since Bitbucket [does not](https://community.atlassian.com/t5/Bitbucket-questions/View-all-comments-on-a-pull-request/qaq-p/677092) show commit comments in the pull request page, the Orb posts a pull request comment if applicable; otherwise it posts a commit comment (only visible in the commit's comments page). If possible, we recommend Bitbucket users to select the "Only build pull requests" option in CircleCI's Project > Advanced settings to get the pull request comments.

The Orb uses the latest version of Infracost by default as we regularly add support for more cloud resources. If you run into any issues, please join our [community Slack channel](https://www.infracost.io/community-chat); we'd be happy to guide you through it.

As mentioned in the [FAQ](https://www.infracost.io/docs/faq), you can run Infracost in your Terraform directories without worrying about security or privacy issues as no cloud credentials, secrets, tags or Terraform resource identifiers are sent to the open-source [Cloud Pricing API](https://github.com/infracost/cloud-pricing-api). Infracost does not make any changes to your Terraform state or cloud resources.

<img src="screenshot.png" width=557 alt="Example screenshot" />

## Parameters

### `path`

**Optional** Path to the Terraform directory or JSON/plan file. Either `path` or `config_file` is required.

### `terraform_plan_flags`

**Optional** Flags to pass to the 'terraform plan' command, e.g. `"-var-file=my.tfvars -var-file=other.tfvars"`. Applicable when path is a Terraform directory.

### `usage_file`

**Optional** Path to Infracost [usage file](https://www.infracost.io/docs/usage_based_resources#infracost-usage-file) that specifies values for usage-based resources, see [this example file](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) for the available options.

### `config_file`

**Optional** Path to the Infracost [config file](https://www.infracost.io/docs/config_file/). Cannot be used with path, terraform* or usage-file flags. See [this example file](https://github.com/infracost/infracost/blob/master/infracost-example.yml) for the available options.

### `percentage_threshold`

**Optional** The absolute percentage threshold that triggers a pull request comment with the diff. Defaults to 0, meaning that a comment is posted if the cost estimate changes. For example, set to 5 to post a comment if the cost estimate changes by more than plus or minus 5%.

## Environment variables

The following environment variables are required. Other supported environment variables are described in the [Infracost docs](https://www.infracost.io/docs/environment_variables).

Terragrunt users should follow [this section](https://www.infracost.io/docs/terragrunt).

### `INFRACOST_API_KEY`

**Required** To get an API key [download Infracost](https://www.infracost.io/docs/#installation) and run `infracost register`.

### `GITHUB_TOKEN`

**Optional** GitHub token used to post comments (e.g. a Personal access token), needs to have `repo` scope so it can post comments.

### `BITBUCKET_TOKEN`

**Optional** Bitbucket "username:password" used to post comments (e.g. "myusername:my_app_password"), the password needs to have Read scope on "Repositories" and "Pull Requests" so it can post comments. Using a [Bitbucket App password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) is recommended.

### Cloud credentials

**Required** You do not need to set cloud credentials if you use Terraform Cloud/Enterprise's remote execution mode, instead you should follow [this section](https://www.infracost.io/docs/terraform_cloud_enterprise).

For all other users, the following is needed so Terraform can run `init`:
- AWS users should set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
- GCP users should set `GOOGLE_CREDENTIALS` or read [this section](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/provider_reference#full-reference) of the Terraform docs for other options.

### `INFRACOST_TERRAFORM_BINARY`

**Optional** Used to change the path to the `terraform binary` or version, see [here](https://www.infracost.io/docs/environment_variables/#cicd-integrations) for the available options.

### `GIT_SSH_KEY`

**Optional** If you're using terraform modules from private Git repositories you can set this environment variable to your private Git SSH key so terraform can access your module.

### `BITBUCKET_API_URL`

**Optional** Bitbucket API URL, defaults to https://api.bitbucket.org.

## Usage

1. In CircleCI, go to your Project Settings > Environment Variables, and add environment variables for `INFRACOST_API_KEY`, either `GITHUB_TOKEN` or `BITBUCKET_TOKEN`, and any other required credentials (e.g. `AWS_ACCESS_KEY_ID`).

2. Create a new file at `.circleci/config.yml` in your repo with the following content. Use the Parameters section above to decide which options work for your Terraform setup. The following example uses `path` to specify the location of the Terraform directory and `terraform_plan_flags` to specify the variables file to use when running `terraform plan`.

  ```
  version: 2.1
  orbs:
    infracost: infracost/infracost@0.5.0
  workflows:
    main:
      jobs:
        - infracost/infracost:
            path: path/to/code
            terraform_plan_flags: -var-file=my.tfvars
  ```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## Publishing

To publish Orb, you can amend the commit or push another commit with [semver:FOO] in the subject where FOO is 'major', 'minor', or 'patch'. CircleCI will automatically bump the Orb version based on the semver commit.

To indicate intention to skip promotion, include [semver:skip] in the commit subject instead.

## License

[Apache License 2.0](https://choosealicense.com/licenses/apache-2.0/)

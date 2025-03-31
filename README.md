<div align="center">
  <a href="https://github.com/iller7/faizal-aws" target="_blank" rel="noopener noreferrer">
    <picture>
      <img width="80" src="docs/images/eks.png" alt="Amazon Elastic Kubernetes Service logo">
    </picture>
  </a>
  <div align="center">
    [![Stars](https://raw.githubusercontent.com/iller7/faizal-aws/refs/heads/main/docs/images/stars.svg)](LICENSE)
    [![License](https://raw.githubusercontent.com/iller7/faizal-aws/refs/heads/main/docs/images/license.svg)](LICENSE)
  </div>
  <h2>Amazon Elastic Kubernetes Service Setup</h2>
</div>

## Setup

There are a number of one-off steps needed to begin with, first we need to create the infra where we will deploy the CloudFront Distribution.

For this, we will connect a github repo to AWS and use the git-sync feature to automatically synchronise the architecture

### Github

Connect your repository using [AWS Connections](https://eu-west-1.console.aws.amazon.com/codesuite/settings/connections). Currently only these services are supported:
- GitHub
- GitHub Enterprise
- GitLab
- Bitbucket
- GitLab self-managed

Create a [new role](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/git-sync-prereq.html#git-sync-prereq-iam) for syncing the stack with a Git repo.
While you're here, create a new service role for cloudformation. Add the AWSCodeDeployRoleForCloudFormation to its permissions policies.

Once the preliminary steps above have been completed, the next steps are:

1. Sign in to the AWS Management Console and open the [AWS CloudFormation](https://console.aws.amazon.com/cloudformation) console.
1. On the navigation bar at the top of the screen, choose the AWS Region to create the stack in.
1. On the Stacks page, choose Create stack, and then choose With new resources (standard).
1. On the Create stack page, do the following:
    - For Prerequisite - Prepare template, keep Choose an existing template selected.
    - For Specify template, choose Sync from Git, and then choose Next.
1. On the Specify stack details page, for Stack name, type a name for your stack. Stack names can include letters (A-Z and a-z), numbers (0-9), and dashes (-).
1. For Stack deployment file, Deployment file creation:
    - If you haven't created a stack deployment file and added it to your repository, choose Create the file using the following parameters and place it in my repository.
    - If you have a stack deployment file in your repository, choose I am providing my own file in my repository.
1. For Template definition repository, choose Choose a linked Git repository to choose a Git repository that was created earlier.
1. In the Repository list, select the Git repository that contains your stack template file.
1. In the Branch list, select the branch you'd like Git sync to monitor.
1. For the Deployment file path, specify the full path including the stack deployment file name from the root of your repository branch.
    - If CloudFormation is generating the file for you, this is where the file will be committed in your repository. If you are providing the file, this is the location of the file in your repository.
1. Add an IAM role. The IAM role includes permissions that are required for CloudFormation to sync the stack from your Git repository. You can choose New IAM role to generate a new role, or choose Existing IAM role to select an existing role from your AWS account. If you choose to generate a new role, the required permissions are included in the role.
1. Enable or turn off comments on pull request:
    - To have CloudFormation post change set information in pull requests for stack updates, keep the Enable comment on pull request toggle switched on.
    - If you switch this toggle off, CloudFormation won't describe the differences between the current stack configuration and the proposed changes in pull requests when the repo files are updated.
1. For the Template file path, specify the full path from the root of your repository for the stack template file.
1. (Optional) To specify the stack parameters, choose Add parameter, provide a key and value for each parameter, and then choose Next. For more information, see Stack deployment file.
1. (Optional) To specify stack tags, choose Add new tag, provide a tag key and value for each tag, and then choose Next. For more information, see Stack deployment file.
1. Choose Next to continue to Configure stack options. For information about configuring stack options, see [Configure stack options](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html#configure-stack-options). When you've completed your stack configuration, choose Next to continue.
1. Review your stack settings and confirm the following:
    - The stack template is configured correctly and set to Sync from Git.
    - The deployment file is configured correctly.
    - The template definition repository is configured correctly, in particular, that the correct Repository and Branch name are selected.
    - The preview of the deployment file is correct and contains the expected parameters and values.
1. Choose Submit to create the stack.

After you choose Submit, a pull request is automatically created in your Git repository. You must merge this pull request into your Git repository to create your stack. After the stack is created, CloudFormation monitors your Git repository for changes.

### Infra

First install the aws cli tool
```sh
brew install awscli
```
If you choose to bypass the sync method above, you can use the cloudformation file in the./clouformation folder to deploy your infra directly using this command:

```sh
aws cloudformation deploy --output table --region eu-west-1 --profile FaizalAWS --stack-name faizal-infra --tags deployed-by=aws-cli --capabilities CAPABILITY_NAMED_IAM --template-file ./cloudformation/cfn.yaml
```
This will create a Cloudfront Distribution with all your networking

## EKS

### Cluster

We will setup the cluster with eksctl, if you need to install the `eksctl` tool then use:
First install the eks cli tool
```sh
brew install eksctl
```
Now apply the the configuration file:
```sh
export EKS_CLUSTER_NAME=faizal-infra
envsubst < ./cluster/eksctl/template.yaml > ./cluster/eksctl/cluster.yaml
eksctl create cluster -f ./cluster/eksctl/cluster.yaml
```
This will take all the environment variables on your system and replace the variables in template.yaml into a new file called cluster.yaml
# Using Github Actions Stages and Shared Workflows

![image](https://github.com/luqmannnn/github-actions-stages/assets/9068525/57e69f4f-4d6b-4429-b27e-cb4a76f37fa6)


## What is Github Actions and why do we use it?
GitHub Actions is a powerful automation tool provided by GitHub, enabling you to automate various software development workflows directly within your repositories. It allows you to define custom CI/CD (Continuous Integration/Continuous Deployment) processes, automate tasks, and respond to various GitHub events.

It's a great tool to centralize your code repositories and run CI/CD processes in.

## Pre-requisites
1. Get a Github Actions token and save it in your secrets to be used in the pipeline. Ref: https://docs.github.com/en/enterprise-server@3.6/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

2. Store all your AWS credentials in Github Actions secrets as well. For this repository, we will not be including them, but you may refer to other examples like [this repo link](https://github.com/luqmannnn/aws-secret) or [this link](https://github.com/aws-actions/configure-aws-credentials)

3. Ensure your stable branches are properly protected with the right [branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule).

4. Ensure your Github Actions is turned on. Go to Settings > Actions > General > Allow all actions and reusable workflows (depends if you'd like to restrict this further)
<img width="1216" alt="Screenshot 2023-12-01 at 4 41 34 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/3bfbd0d7-9a17-4efe-8cf7-ac985e7abe00">


# Here's what we can explore

## Single automated workflow
Let's say you want to build a single CI/CD yaml file for everything - your pull requests, your merged codes etc. 

You can use one single yaml file to define all these steps. You can use [conditionals](https://docs.github.com/en/actions/using-jobs/using-conditions-to-control-job-execution) to allow jobs to trigger ONLY if they meet certain requirements e.g.
1. Only run a deploy job if your branch is 'DEV' branch and NOT a pull_request branch.
2. Only trigger unit testings and package vulnerability scanning on lower environment branches e.g. 'feature/**', 'DEV' and 'UAT' but NOT 'PROD'

To get started, you may refer to the file (combined.yaml) in the ga_template_files/single_workflow folder and notice how we specify the 'on' statements at the top to pick up multiple branches as well as the 'if' statements at the job level.

What happens here is that a Pull Request will only run the first job (env-print)
<img width="1431" alt="Screenshot 2023-12-01 at 4 18 35 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/57230678-493b-414f-927a-c7c252464798">


After the Pull Request is merged, the code will automatically be merged from DEV --> UAT --> PROD
<img width="1429" alt="Screenshot 2023-12-01 at 4 18 53 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/381aa1c8-ef8d-4a94-a187-74466c47b358">

This is because a Merged Pull Request will have the same effect as a 'ON: PUSH' to 'DEV' branch.

## Multiple automated workflows
Let's say now you want to have granular CI/CD yaml files for each environment, maybe even for your pull request branches.

You can break down a single CI/CD yaml file into multiple yaml files and better control what gets triggered at each stage. For this, you may refer to the files in the ga_template_files/multiple_workflows folder.

What happens here initially is a simple Pull Request, which triggers an environment print without merging any code.
<img width="1435" alt="Screenshot 2023-12-01 at 4 31 31 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/44d0bec6-8335-4ba8-b1f7-beec4901fd16">

Afterwards, once the Pull Request is merged, the code will flow from branch --> DEV --> UAT --> PROD automatically. Take a look at the latest 4 jobs in the below screenshot.
<img width="1434" alt="Screenshot 2023-12-01 at 4 33 44 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/125c72bf-177d-46af-a9eb-543482673618">

A detailed look at the closure of the Pull Request, which merges the code to DEV and automatically merges to UAT.
<img width="1433" alt="Screenshot 2023-12-01 at 4 34 04 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/0ba284dc-34de-41e5-8c42-1efaa6bc278a">

A detailed look at the UAT to PROD merge.
<img width="1431" alt="Screenshot 2023-12-01 at 4 34 25 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/ce311271-3b52-4d1f-be38-578018992360">

And finally a detailed look at what happens after the PROD code has been updated.
<img width="1437" alt="Screenshot 2023-12-01 at 4 34 42 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/fb7f19af-7cc4-4394-8177-c0ca183d989e">

## Inputs in your workflow
Let's say now you want to trigger a CI/CD workflow which requires inputs that can be used in your pipeline. You can do this using [workflow_dispatch](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/) > [inputs](https://github.blog/changelog/2021-11-10-github-actions-input-types-for-manual-workflows/)

For this, you may refer to the files in the ga_template_files/input_workflows folder.

Under the Actions tab, after clicking the input workflow from the left side of the page, you can see a dropdown list button calle "Run workflow" on the right. Clicking that will give you the opportunity to provide inputs as listed in the yaml file.

<img width="1436" alt="Screenshot 2023-12-01 at 4 48 58 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/535a130a-449f-485e-a2d5-199ff9b79e3c">

Let's try providing some inputs like that below and running the workflow.

<img width="362" alt="Screenshot 2023-12-01 at 4 50 09 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/0c33e5f9-e899-4888-a0b3-3d3f20fec1a6">

And now you are able to use the inputs in your code.
<img width="1429" alt="Screenshot 2023-12-01 at 4 54 42 PM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/fe0090c1-ce90-4e1e-a8b4-51d7fbb40067">


## Reusing another workflow
Let's say now you have 5 different microservices that needs to run the same steps e.g. same deployment steps or same automation steps. Instead of copying the same steps across the 5 different repositories' workflow files, you can create a reusable workflow in one repository and have your 5 repositories call the [reusable workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows).

For this, you may refer to the files in the ga_template_files/shared_workflows folder.

The reusable workflow in this case can be found here: https://github.com/luqmannnn/reusable-workflow/blob/main/.github/workflows/reusable-workflow.yaml. We can see that it requires 3 different inputs:
1. user name
2. address
3. age

We will need to define these in our shared_workflow file or get inputs from user before triggering.

If successful, it would look something like this:
<img width="1435" alt="Screenshot 2023-12-02 at 12 34 32 AM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/b20d5f29-bd6c-46f9-8aa1-f9b62daa5b3b">

We can see the variables passed from our workflow to the shared workflow:
<img width="1434" alt="Screenshot 2023-12-02 at 12 35 12 AM" src="https://github.com/luqmannnn/github-actions-stages/assets/9068525/c647f823-fbf7-4a29-8875-44d0a5d339b9">


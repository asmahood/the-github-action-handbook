# Introduction to Github Actions

## Types of Github Actions

There are three types of github actions:

1. Container Actions
2. JavaScript Actions
3. Composite Actions

### Container Actions
With **container actions**, the environment is part of the action's code. These actions can only be run in a Linux environment that Github hosts. Container actions support many different languages.

### JavaScript Actions
**JavaScript actions** don't include the environment in the code. You'll have to specify the environment to execute these actions. YOu canr un these actions in a VM in the cloud or on-premises. JavaScript actions support Linux, macOS, and Windows environments.

### Composite Actions
**Composite Actions** allow you to combine multiple workflow steps within one action. For example, you can use this feature to bundle together multiple run commands into an action, and then have a workflow that executes the bundled commands as a single step using that action.

## The anatomy of a Github Action
Below is an example of an action that performs a git checkout of a repository and builds Node.js code that was checked out. This action uses another action `actions/checkout@v1` as part of a step in a workflow.

```yml
steps:
  - uses: actions/checkout@v1
  - name: npm install and build webpack
    run: |
      npm install
      npm run build
```

Suppose you want to use a container action to run containerized code. Your action might look like this:

```yml
name: "Hello Actions"
description: "Greet someone"
author: "octocat@github.com"

inputs:
    MY_NAME:
      description: "Who to greet"
      required: true
      default: "World"

runs:
    uses: "docker"
    image: "Dockerfile"

branding:
    icon: "mic"
    color: "purple"
```

The `inputs` section describes the variables that need to be set in the workflow that run this action.

The `runs` section, we specify Docker in the `uses` attribute. When you do this you'll need to provide the path to the Docker image file. Here, it's called Dockerfile.

The last section `branding` personalizes your action in the Github Marketplace if you decide to publish it.

## What is a Github Action Workflow?
A GitHub Actions workflow is a process that you set up in your repository to automate software-development lifecycle tasks, including GitHub Actions. With a workflow, you can build, test, package, release, and deploy any project on GitHub.

A workflow must have at least **job**. A job is a section of the workflow associated with a **runner**. 

A runner can be Github-hosted or self-hosted, and the job can run on a machine or in a container. You'll specify the runner with the `runs-on` attribute.

### Github-hosted versus self-hosted runners

A runner is simply a server that has the Github actions runner application installed. When it comes to runners there are two options to choose from. If you use a Github-hosted runner, each job runs in a fresh instance of a virtual environment. The Github-hosted runner type you define (`runs-on: {operating system-version}`) specifies that environment. With self-hosted runners, you need to apply the self-hosted label, its operating system, and the system architecture. For exampe, a self-hosted runner with a Linux operating system and ARM32 architecture would look like: `runs-on: [self-hosted, linux, ARM32]`.

Github-hosted runners offer a quicker and simpler way to run your workflows, but with limited options.

Self-hosted runners are a highly configurable way to run workflows in your own custom local environment. You canr un self-hosted runners on-premises or in the cloud. You can also use self-hosted runners to create a custom hardware configuration with more processing power or memory.

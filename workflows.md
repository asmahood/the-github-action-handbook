# Workflows

## Table of Contents
- [The Components](#the-components)
  - [Workflows](#workflows)
  - [Jobs](#jobs)
  - [Steps]($steps)
  - [Actions](#actions)
- [Creating a Workflow](#creating-a-workflow)
- [Configuring a Workflow](#configuring-a-workflow)
  - [Configuring workflows to run on a schedule](#configure-workflows-to-run-on-a-schedule)
  - [Configuring workflows to run for manual events](#configuring-workflows-to-run-for-manual-events)
  - [Configuring workflows to run for webhook events](#configuring-workflows-to-run-for-webhook-events)
  - [Workflow permissions](#workflow-permissions)
- [Use conditional keywords](#use-conditional-keywords)
- [Disable and Delete Workflows](#disable-and-delete-workflows)
- [Configuring an Action](#configuring-an-action)
  - [Use specific versions of an action](#use-specific-versions-of-an-action)

## The Components

![Github Action Workflow Components](./img/github-actions-workflow-components.png)

In short, an event triggers a **workflow** which contains a **job**. The job then uses **steps** to dictate which **actions** wull run within the workfow.

### Workflows

A workflow is an automated process that you add to your repository. A workflow needs at least *one* job, and different events can trigger it. It can be used to build, test, package, release, or deploy a project.

### Jobs

The job is the first major component within a workflow. A job is a section of the workflow that will be associated with a runner. A job can run on a machine or in a container. You'll specify the runner with the `runs-on` attribute.

### Steps
A step is an individual task that you can run commands in a job.

### Actions
The actions insde a workflow are the standalone commands that are executed. These standalone commands can reference Github Actions such as using your own custom actions, or community actions. You can also run commands such as `run: npm install -g bats` to execute a command on the runner.

## Creating a Workflow

To create a workflow, start by adding a new `.yml` file in the `.github/workflows` directory of your repository. Below is an example of a workflow:

```yml
name: A workflow for my Hello World file # The name that will apear in the Actions tab of a repo
on: push
jobs:
  build:
    name: Hello world action
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: ./action-a
      with:
        MY_NAME: "Mona"
```

## Configuring a Workflow

Below are a few common configurations of a workflow file as well as some different ways to configure a workflow and best practices.

### Configure workflows to run on events

The `on` attribute specifies what will trigger this workflow to run. Here, it triggers whenever there's a push event to the repository. You can specify single events like `on:push`, an array of events like `on: [push, pull_request]`, or an event-configuration map that schedules a workflow or restricts the execution of a workflow to specific files, tags, or branch changes:

```yml
on:
  # Trigger the workflow on push or pull request,
  # but only for the main branch
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  # Also trigger on page_build, as well as release created events
  page_build:
  release:
    types: # This configuration doesn't affect the page_build event above
      - created
```

[A full list of triggerable events is listed here.](https://docs.github.com/actions/using-workflows/events-that-trigger-workflows)


### Configure workflows to run on a schedule

The `schedule` event allows you to trigger a workflow to run at specific UTC times using POSIX cron syntax. For example, if you want to run a workflow every 15 minutes:

```yml
on:
  schedule:
    - cron: '*/15 * * * *'
```
If you want a workflow to run every sunday at 3:00am:
```yml
on:
  schedule:
    - cron: '0 3 * * SUN'
```
The shortest interval you can run scheduled workflows is once every 5 minutes, and they run on the latest commit on the default or base branch.

### Configuring workflows to run for manual events

You can manually trigger a workflow by using the `workflow_dispatch` event. This event allows you to run the workflow by using the Github REST API or by selecting the **Run workflow** button in the **Actions** tab within a repository. Using `workflow_dispatch`, you can choose on which branch you want the workflow to run, as well as set optional `inputs` that Github will present as form elements in the UI:

```yml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'     
        required: true
        default: 'warning'
      tags:
        description: 'Test scenario tags'  
```

In addition to `workflow_disptach`, you can use the Github API to trigger a webhook event called `repository_dispatch`. This event allows you to trigger a workflow for activity that occurs outside of Github. It essentially serves as an HTTP request to your repository asking Github to trigger a workflow off an action or webhook. Using this manual event requires you to do two things: send a `POST` request to the GitHub endpoint `/repos/{owner}/{repo}/dispatches` with the webhook event names in the request body, and configure your workflow to use the `repository_dispatch` event:

```bash
curl \
  -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/octocat/hello-world/dispatches \
  -d '{"event_type":"event_type"}'
```
```yml
on:
  repository_dispatch:
    types: [opened, deleted]
```

### Configuring workflows to run for webhook events
Lastly, you can trigger a workflow to run when specific webhook events occur on Github. YOu can trigger most webhook events from more than one activity for a webook. If multiple activities exist for a webhook, you can specify an activity type to trigger the workflow. For example, you can run a workflow for the `check_run` event, but only for the `rerequested` or `requested_action` activity types:

```yml
on:
  check_run:
    types: [rerequested, requested_action]
```

### Workflow permissions

You can give your workflow permissions to operate on the repository. Consider the example below:

```yml
name: Post welcome comment
on:
  pull_request:
    types: [opened]
permissions:
  pull-requests: write
```

This gives the workflow permission to write to pull requests.

## Use conditional keywords
Within a workflow file, you can access context information and evaluate expressions. Expressions are communly used with the conditional `if` keyword in a workflow file to determine whether a step should run or not. You can also use any supported context and expression to create a conditional. It's important to know that when using conditionals in a workflow, you need to use the specific syntax `${{ <expression> }}. 

For example, a workflow uses the `if` conditional to check if the `github.ref` (the branch or tag ref that triggered the workflow run) matches `refs/head/main`:

```yml
name: CI
on: push
jobs:
  prod-check:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      ...
```

Notice that in this example, the `${{ }}` are missing from the syntax. With some expressions, as in the case of the `if` conditional, you can omit the expression syntax. Github automatically evaluates some of these common expressions, but you can always include them in case you forget which expressions Github automatically evaluates.

## Disable and Delete Workflows
You can stop a workflow from being triggered without having to delete the file from the repo, either on GitHub or through the GitHub REST API. When you wish to enable the workflow again, you can easily do it using the same methods.

![Disable workflow in repo](./img/disable-workflow.png)

You can also cancel a workflow run that's in progress in the Github UI from the Actions tab or by using the Github API endpoint `DELETE /repos/{owner}/{repo}/actions/runs/{run_id}`. Keep in mind that when you cancel a workflow run, Github will cancel all of its jobs and steps within that run.

<table width="100%">
<tr>
  <td align="left"><a href="introduction.md">Previous: Introduction</a></td>
  <td align="right"><a href="">Next: </a></td>
</tr>
</table>

## Configuring an Action

### Use specific versions of an action

When referencing actions in your workflow, we recommend that you refer to a specific version of that action rather than just the action itself. By referencing a specific version, you're placing a safeguard from unexpected changes pushed to the action that could potentially break your workflow. Here are several ways you can reference a specific version of an action:

```yml
steps:    
  # Reference a specific commit
  - uses: actions/setup-node@c46424eee26de4078d34105d3de3cc4992202b1e
  # Reference the major version of a release
  - uses: actions/setup-node@v1
  # Reference a minor version of a release
  - uses: actions/setup-node@v1.2
  # Reference a branch
  - uses: actions/setup-node@main
```

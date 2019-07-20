---
title: "GitHub Actions"
date: 2019-07-20T23:37:35+03:00
draft: false
toc: false
images:
tags:
  - github
  - actions
  - github actions
---

Hello everyone, how is it going? Today, I will introduce to you GitHub Actions.

Recently, I changed one of our project's CI/CD pipeline to GitHub Actions from CircleCI. This is a unique experience for me. So let's get started.

As you know well, GitHub Actions still on the beta. There may bugs. Github Actions is a service for automating your projects. Like a CircleCI or Jenkins. You can build CI/CD pipelines for your projects. You can create multiple workflows. Let's create one step by step.

- Go to Actions on your repo

![SS1](/static/images/github-actions/ss1.png)

- Create a new workflow

![SS2](/static/images/github-actions/ss2.png)

- Here is what you should see

![SS3](/static/images/github-actions/ss3.png)

Now, let's suppose we have a NodeJS package. We want to build and deploy it on every push on the master branch. You can build workflow with a visual editor but my choice is code-based.

```
workflow "New workflow" {
  resolves = [
    "Deploy"
  ]
  on = "push"
}
```

Resolves means which actions need to be run

First, we need a filter for branch

```
action "Master Branch Filter" {
  uses = "actions/bin/filter@master"
  args = "branch master"
}
```

Now, we can build the package

```
action "Build" {
  uses = "actions/npm@master"
  needs = "Master Branch Filter"
  runs = "npm"
  args = "install"
}
```

Finally, we can deploy the package

```
action "Deploy" {
  uses = "actions/npm@master"
  needs = "Build"
  runs = "npm"
  args = "publish --access public"
  secrets = ["NPM_AUTH_TOKEN"]
}
```

As you can see, there is a `secrets` and `needs` field on Deploy Action. You can define secrets on `Settings > Secrets` page. Needs field means this action need to be run `Build` action

Final Code
```
workflow "Node Package Pipeline" {
  resolves = [
    "Deploy"
  ]
  on = "push"
}

action "Master Branch Filter" {
  uses = "actions/bin/filter@master"
  args = "branch master"
}

action "Build" {
  uses = "actions/npm@master"
  needs = "Master Branch Filter"
  runs = "npm"
  args = "install"
}

action "Deploy" {
  uses = "actions/npm@master"
  needs = "Build"
  runs = "npm"
  args = "publish --access public"
  secrets = ["NPM_AUTH_TOKEN"]
}
```

### Conclusion

After all, this is a good experience for me who works as a software developer. But there is not enough documentation for more complex workflows. There are several predefined actions. You can check it out in this link.

https://github.com/features/actions

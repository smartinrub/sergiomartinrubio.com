---
title: Automated Dependency Updates with Dependabot
image: https://lh3.googleusercontent.com/-yzcoB3i-TKOeiKkity2ECG-2lCb0ltTC4u57egckupmxjU5Yi_bGltbj68goA6V856430GWcABwFl8X1gkDoKaaR85ShfFD9XKi1fOGBuRGqD9Br-XGYitQkrBG7J60NuKM6SAV_PcPRPzta_zmAEm7U8rl4pHQe9Nlgkh33Pug5pZZ2jCyDwfX_8lRiBiaoSO3bo1dYW-3IKXv_zxNnD03K8ppGFM2oWGmKF7tCzhz0VV7GyJaazHPY2lODcaoOna1tKm6HuQHcoQF53DFDy5x22KuRSrgDIvJIfRckhtn33vdgsGvBWS6vgPAWYDCs4rGC6vcS2s7AxFxR035JC2GCiMSbqs7WWCqkQ3mJIhwq5aj1VXvk51rx6-JbbseHOOxB0_M3AYXUhq3FQE82sn4YBWbZAUsktb5jxt8wa9Mz7kLrvHy8sclsAs7IwEjkds5Ye9A5szhGX1j_JF5_J_CmYIMy7NmryHyQV4thOI_6dv4BheViqOZd5wDfZBR0qjo95it5BMNh91lDMjukiVxyvSqJRn-Jf2FyYskIUjTziAWg2_wwapEAmv9zvfZO7j9U4jDKe4UvZrytBzw4VfXN8H-QgApeJQRYqc6X7AUQhXcQOEhfZQY2DwErI0YXLW-DjvHk3ABLXWNghKwLO6vheP7v3c9ENa-MHrsWf5xQpgLlZCP3W-BER_0=w640-h425-no?authuser=0
author: Sergio Martin Rubio
categories:
    - DevOps
    - Version Control
mermaid: false
layout: post
---

## Introduction

[Dependabot](https://dependabot.com){:target="_blank"} is a free and open source tool that has been bought by _GitHub_ and allows you to check for updates on you dependency files.

It can be used to update dependencies in _Ruby_, _JavaScript_, _Python_, _PHP_, _Elixir_, _Elm_, _Go_, _Rust_, _Java_ and _.NET_, as well as, git submodules, _Docker_ files and _Terraform_ files.

## Package Manger Options

- _bundler_
- _pip_ (includes pipenv)
- _npm_and_yarn_
- _maven_
- _gradle_
- _cargo_
- _hex_
- _composer_
- _nuget_
- _dep_
- _go_modules_
- _elm_
- _submodules_
- _docker_
- _terraform_

## GitHub Integration Guide

Dependabot is fully integrated with GitHub and only in a few steps you will be able to add it to your current GitHub projects.

1. Go to [Dependabot site](https://dependabot.com){:target="_blank"} or [GitHub marketplace and search for Dependabot](https://github.com/marketplace?utf8=%E2%9C%93&query=dependabot){:target="_blank"}.
2. Install Dependabot to your GitHub account.
    ![alt text](https://lh3.googleusercontent.com/2LF49w-QRKtX7eEMkjZM83Dn6eJVvdKblx7dRomTj9QKBZ7zZHIf2BKDohN-1eXZmUU4DJUeibuE2FhSeg=w600-no-tmp.jpg "Install Dependabot GitHub")
3. Give permission to all your repositories or selected ones.
    ![alt text](https://lh3.googleusercontent.com/w-nGjPm9AMsVfvkcbSYhu9j482Z55OUdrhOSA6tFupSXRFv9tTD2f2ZAFn6AHRGzLC_MMDGHGK4t-aj8Fg=w600-no-tmp.jpg "Install Dependabot Preview")
4. Start adding repositories to the Dependabot dashboard.
    ![alt text](https://lh3.googleusercontent.com/j4edb8TzKBqykrV_j62qvPQ0ME5NCH3YCD--UynTcIwPdHzUZdxJEshexS-4IdIkX5Evil88lN_tGXNP-Q=w600-no-tmp.jpg "Add Repos Dependabot")
5. Tweak Dependabot settings for your projects: how frequent it runs (daily, weekly...), directory, branch, GitHub Pull Requests info...
    ![alt text](https://lh3.googleusercontent.com/WTi5NcbLw96prl8jRf9HXQ9FJJRmWQVJsddeGyLXG_5Asqcdw9Vqdv5-esoIN_L1tb7Ps92MLW27zOUkyw=w800-no-tmp.jpg "Depedabot Project Settings")
6. Run Dependabot and review PR. A relevant branch name, PR title, comment and labels are created.
    ![alt text](https://lh3.googleusercontent.com/cnB2V_3V22hqUt4vH57167KfX-bKA7Z75523xiZfIdwbvGnx5ykJNBaHiqGLDHRujoVP_nUSZ7-9s5C6kw=w800-no-tmp.jpg "Depedabot Pull Request")

### Breaking Changes

_Dependabot_ also provides a way to give you more confidence in case of possible breaking changes. From the PR page a link is shown and it will take you to a page where you can see PRs that failed CI.

{% include elements/figure.html image="https://lh3.googleusercontent.com/SmtZ5Ujm1c9x6L5Tp1rm3frPr8nn9PchqU1VD0Y5nmMtew5vwvVRU4NHN9t_czdbaSyVVhC7-QqBwA51QA=w800-no-tmp.jpg" caption="Dependabot Compatibility Rate" %}

How does it work?

- _Dependabot_ retrieves metrics of similar updates from other repos.
- It creates a score based on each repo that runs a CI with passed test suite.

{% include elements/figure.html image="https://lh3.googleusercontent.com/BeLflCYfw-UbUS4qOro8GMjzoAJGE44bYhztT6peQp_t7c1g0WpByBgI6MKtos2wEtkYbC8H1BMvXUYqTA=w800-no-tmp.jpg" caption="Compatibility Score Page" %}

>Note: private repositories are not included on the score calculation.

However, to be 100% sure that you are not introducing breaking changes with the dependency updates, you should create a build with the new branch created by _Dependabot_.

## Advance Integrations

**Dependabot** can be also used by itself. The official repository provides a [_Ruby_ script](https://github.com/dependabot/dependabot-script) that you can run with _Docker_. Another option is to create a repository dedicated to the script and configure a _GitLab CI_ pipeline to run it periodically.

For instance, you could setup a **GitLab CI** with an **Azure** repo by following the next steps:

1. Clone the [dependabot-script repo](https://github.com/dependabot/dependabot-script.git){:target="_blank"} and create a new project on **GitLab**
2. Rename `.gitlab-ci.example.yml` to `.gitlab-ci.yml`
3. Get the values of the required global variables: 
    - `AZURE_ACCESS_TOKEN` -> Go to your _Azure Devops profile_, _Security_, _Personal access tokens_, _+ New Token_
    - `PROJECT_PATH` -> For _https://dev.azure.com/econsergio/_git/spring-boot-demo_ will be `econsergio/_git/spring-boot-demo`
    - `PACKAGE_MANAGER_SET` -> _maven_ for this example
4. Create a new pipeline schedule -> Go to _CI/CD_, _Schedules_
5. Set in the schedule the required variables
6. The script creates a branch and PR on _Azure_

{% include elements/figure.html image="https://lh3.googleusercontent.com/f-IlBiczAsWjFzgZahHugQ4vLGsD3qUTChuJl1vCT11IYsWJ9ysoJrP5sjXYD7VDUc7y3aMz1EctSw_QVQ=w800-no-tmp.jpg" caption="Azure Dependabot Branch" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/MRbvNKE_m5SkUZY5wgkEup_6tf0Vzu23rEI8cEsta-FzIeYMf8Tz7zeaBVwa7wjP1M9UHul3C1-lv5pq4Q=w800-no-tmp.jpg" caption="Azure Dependabot Dependency Diff Updates" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/KK1HqPnX3wTNFH2qT5ljWRjmDMWn1UJw6dBaeawaNYklHd7F27sjrZn_A27rNu-xrsk_Mov6ggkkqQC1kQ=w800-no-tmp.jpg" caption="Azure Dependabot PR" %}

## Conclusion

As you can see _Dependabot_ is very flexible and will take care of all your projects dependency updates. Despite it has been acquired by _GitHub_, you can run it on _GitLab_ and point to projects on **Azure** or **GitLab**.

mainlycode/gitlab-runner-docker
===============================

Ansible role for installing and configuring and registering GitLab Runner as a docker container.

# Requirements

This role uses the Ansible [docker modules](http://docs.ansible.com/ansible/latest/guide_docker.html) which [requires](http://docs.ansible.com/ansible/latest/guide_docker.html#requirements)
`docker-py` to be installed.

# Configuration

In order to register the runner with GitLab you need to obtain a registration token from
the runner settings page: https://gitlab.com/[your group]/[your project]/settings/ci_cd.

Next you need to register the token in the {{ gitlab_runner_registration_token }} variable.

For example in your `group_vars/all.yml` file:

```yaml
---
gitlab_runner_registration_token: "yolotrololo"

```

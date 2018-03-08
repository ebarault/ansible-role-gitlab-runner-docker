# Ansible role for dockerized Gitlab runners (docker executor)

Ansible role for installing, configuring and registering a dockerized Gitlab runner as a docker executor.

## Requirements

This role uses the Ansible [docker modules](http://docs.ansible.com/ansible/latest/guide_docker.html) and installs
[`docker-py`](http://docs.ansible.com/ansible/latest/guide_docker.html#requirements).

## Gitlab Configuration

In order to register the runner with GitLab you need to obtain a registration token from
the runner settings page in your Gitlab server.

## Usage

- **gitlab-runner.yml**

```yml
---

- name: Gitlab runner installation
  hosts: my_hosts
  become: yes

  vars:
    gitlab_runner_data_volume: /data/gitlab-runner
    gitlab_runner_image_tag: alpine-v10.3.1
    gitlab_runner_name: docker-shared
    gitlab_runner_gitlab_url: https://gitlab.com/
    gitlab_runner_registration_token: WuT4fioAfrK3PG_hVefy
    gitlab_runner_default_image: docker:18
    gitlab_runner_concurrent: 10 # global to all runners in a same host
    gitlab_runner_limit: 0 # per token, 0 means unlimited
    gitlab_runner_request_concurrency: 10 # per runner
    gitlab_runner_tags: docker
    gitlab_runner_run_untagged: false

  tasks:
    - name: Call gitlab-runner role with vars
      include_role:
         name: gitlab-runner
      vars:
        data_volume: "{{gitlab_runner_data_volume}}"
        image_tag: "{{gitlab_runner_image_tag}}"
        runner_name: "{{gitlab_runner_name}}"
        gitlab_url: "{{gitlab_runner_gitlab_url}}"
        registration_token: "{{gitlab_runner_registration_token}}"
        default_image: "{{gitlab_runner_default_image}}"
        concurrent: "{{ gitlab_runner_concurrent }}"
        limit: "{{gitlab_runner_limit}}"
        request_concurrency: "{{gitlab_runner_request_concurrency}}"
        runner_tags:  "{{gitlab_runner_tags}}"
        run_untagged: "{{gitlab_runner_run_untagged}}"
        tlsverify: "{{gitlab_runner_tlsverify}}"
```

- **requirements.yml**

```yml
# Install gitlab-runner role from github
- name: gitlab-runner
  src: https://github.com/ebarault/ansible-role-gitlab-runner-docker
  version: "1.2.1"
```

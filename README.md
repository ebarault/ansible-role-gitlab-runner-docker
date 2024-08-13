# Ansible role for dockerized Gitlab runners (docker executor)

Ansible role for installing, configuring and registering a dockerized Gitlab runner as a docker executor.

## Requirements

This role uses the Ansible [docker modules](http://docs.ansible.com/ansible/latest/guide_docker.html) and installs
[`docker`](http://docs.ansible.com/ansible/latest/guide_docker.html#requirements).

It expects docker to be installed on the target host.

## Registering Gitlab Runners 

see: https://docs.gitlab.com/runner/register/

## Usage

- **gitlab-runner.yml**

```yml
---

- name: Gitlab runner installation
  hosts: my_hosts
  become: yes

  vars:
    gitlab_runner_data_volume: /data/gitlab-runner
    gitlab_runner_image: public.ecr.aws/gitlab/gitlab-runner:alpine3.19-v17.1.1
    gitlab_runner_name: docker-shared
    gitlab_runner_gitlab_url: https://gitlab.greenflex.com
    gitlab_runner_auth_token: xxxxxxxxxx
    gitlab_runner_default_image: public.ecr.aws/docker/library/alpine:3.20.2
    gitlab_runner_concurrent: 10
    gitlab_runner_limit: 0
    gitlab_runner_request_concurrency: 10
    gitlab_runner_metrics_listen_address: ":9252"

  tasks:
    - name: Call gitlab-runner role with vars
      include_role:
         name: gitlab-runner
      vars:
        data_volume: "{{gitlab_runner_data_volume}}"
        image: "{{gitlab_runner_image}}"
        runner_name: "{{ansible_hostname}}"
        gitlab_url: "{{gitlab_runner_gitlab_url}}"
        runner_auth_token: "{{gitlab_runner_auth_token}}"
        default_image: "{{gitlab_runner_default_image}}"
        concurrent: "{{gitlab_runner_concurrent}}"
        limit: "{{gitlab_runner_limit}}"
        request_concurrency: "{{gitlab_runner_request_concurrency}}"
        metrics_listen_address: "{{gitlab_runner_metrics_listen_address}}"
```

- **requirements.yml**

```yml
# Install gitlab-runner role from github
- name: gitlab-runner
  src: https://github.com/ebarault/ansible-role-gitlab-runner-docker
  version: "alpine3.19-v17.1.1"
```

Please refer to [default/main.yml](https://github.com/ebarault/ansible-role-gitlab-runner-docker/blob/master/defaults/main.yml) for parameters documentation.
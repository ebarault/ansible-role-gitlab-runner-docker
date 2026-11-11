# Ansible role for dockerized Gitlab runners (docker executor)

Ansible role for installing, configuring and registering a dockerized Gitlab runner as a docker executor.

## Requirements

This role uses the Ansible [docker modules](http://docs.ansible.com/ansible/latest/guide_docker.html) and installs
[`docker`](http://docs.ansible.com/ansible/latest/guide_docker.html#requirements).

It expects docker to be installed on the target host.

## Registering Gitlab Runners 

see: https://docs.gitlab.com/runner/register/

## Warning: destructive behavior on every run

Applying this role is destructive: it always removes the local runner configuration and, depending on how the runner was created, may delete the runner itself on the GitLab server. Every playbook run does the following:

1. **Recreates the runner container**: the `docker_container` task is declared with `recreate: yes` and `restart: yes`, so the container is torn down and recreated on each run. Any job currently handled by the container is killed, and the container is restarted a second time after `config.toml` has been updated.

2. **Unregisters every runner known to the container, unconditionally**: `gitlab-runner unregister --all-runners` runs on every run, before any registration. It unregisters all runners listed in the mounted `config.toml` — including any additional runner you may have registered manually in the same volume. Per the [official GitLab Runner documentation](https://docs.gitlab.com/runner/commands/#gitlab-runner-unregister), this operation cannot be undone and always removes the runner entries from the local `config.toml`. What happens on the GitLab server depends on how each runner was created:

   - runner created with a **runner registration token**: unregistering with the runner authentication token **deletes the runner** — the runner record, its runner ID, its tags and its assignments are permanently lost on the GitLab server;
   - runner created **in the GitLab UI or with the Runners API** (the current flow, which issues runner authentication tokens): unregistering **deletes the runner manager, but not the runner** — the runner itself, its tags and its assignments are kept on the GitLab server, and the role re-registers a new runner manager against it using the same authentication token.

3. **Re-registers the runner from scratch**: the register command sets only the parameters passed to the role (GitLab URL, token, executor, default image, name, limits). It does not use `--tag-list` nor `--leave-runner`, so runner-level settings such as tags are neither set nor restored by the role — they remain managed on the GitLab server side for runners created in the UI or with the Runners API, and are lost with the runner for runners created with a legacy registration token.

If the register step fails after the unregister step has run, the host is left with no working runner until a later successful run.

Recommended precautions:

- Run the playbook when **no jobs are running** on this runner: in-progress jobs are killed by the container recreation.
- **Back up `config.toml`** (the `data_volume` directory mounted on `/etc/gitlab-runner`, default `/data/gitlab-runner/config.toml`) before running: it is the only remaining record of the runners removed by the unregister step. The [GitLab Runner documentation](https://docs.gitlab.com/runner/commands/#gitlab-runner-unregister) explicitly recommends backing up `config.toml` before this operation.
- Plan for **token rotation** if your runner was created with a legacy registration token: the unregister step deletes that runner on the GitLab server, so be ready to create a new runner and generate a new authentication token. Runners created in the GitLab UI or via the Runners API keep their runner record and authentication token across runs, so re-registration with the same token should succeed.
- Avoid registering **additional runners in the same volume**: `--all-runners` removes all of them, not only the one managed by this role — their local configuration is always lost, and runners created with a legacy registration token are deleted on the GitLab server.

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
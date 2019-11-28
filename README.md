# Ansible Role javascript-app

Library of Ansible plugins and roles for deploying various services.
See [ansible-roles](https://github.com/davidfischer-ch/ansible-roles) for additional documentation.

This repository hosts the role javascript-app and may depend of other roles and plugins of the library.

This role streamline operating a JavaScript front-end application (cloning, building, uploading, ...).

## Dependencies

See [meta](meta/main.yml).

## Example

### Project

Add to your front-end application the following `config.yml` (to adapt based on your project requirements and structure):

```
---

node_version: 10.17.0  # Versions to use to build your application
npm_version: 6.11.3    # See https://nodejs.org/en/download/releases/
source_path: .         # Where the source code reside, typically . or src/
static_path: dist/     # See https://www.quora.com/What-is-dist-folder-in-Angular-application
templates: []          # Path to some jinja2 templates to render before building the application
```

### Playook

```
---

- hosts:
    - some-build-server
  roles:
    - javascript-app
  vars:
    python_pip_environment:

      HTTP_PROXY: http://proxy.your-corp.com:3128
      HTTPS_PROXY: http://proxy.your-corp.com:3128
      NO_PROXY: localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16

      http_proxy: http://proxy.your-corp.com:3128
      https_proxy: http://proxy.your-corp.com:3128
      no_proxy: localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16

      PIP_INDEX_URL: https://nexus.your-corp.com/repository/pypi-proxy/simple
      PIP_TRUSTED_HOST: nexus.your-corp.com

    jsapp_node_environment:

      HTTP_PROXY: http://proxy.your-corp.com:3128
      HTTPS_PROXY: http://proxy.your-corp.com:3128
      NO_PROXY: localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16

      PATH: "{{ jsapp_node_env_paths|join(':') }}"

    jsapp_hosting_implementation: none
    jsapp_instance_name: my-awesome-app

    jsapp_repository_url: git@gitlab.your-corp.com:A-PROJECT/my-awesome-angular-or-react-or-vue-app.git
    jsapp_repository_host: gitlab.your-corp.com
    jsapp_repository_key: gitlab.your-corp.com ssh-rsa AAA(...)E16v
    jsapp_ssh_key_private: '/some/path/repository_rsa'
    jsapp_ssh_key_public: '/some/path/repository_rsa.pub'
    jsapp_version: production
```

## Variables

### Required

You must set the value of those variables ...

#### Variable `jsapp_hosting_implementation`

Any valid value in `jsapp_hosting_implementations`:

- `none`: Do nothing.
- `s3`: Upload on an AWS S3 bucket.

If hosting on a S3 bucket, then you must define extra variables (see [Required If Hosting On AWS S3](#required-if-hosting-on-aws-s3)).

#### Variable `jsapp_instance_name`

Name your application as you wish.

#### Variable `jsapp_repository_url`

The application source code can be either copied from the control machine (is a path) or cloned using git module (is an URL).

If cloning from a Git repository, then you must define extra variables (see [Required If Cloning From Git](#required-if-cloning-from-git)).

You can define the value of `jsapp_repository_is_local` if the auto-detection fail.

#### Variable `jsapp_role_action`

This role implements multiple actions related to managing a JavaScript front-end application.
This variable predated the import_role's tasks_from option, but in a more controlled manner.

Any valid value in `jsapp_role_actions`:

- `run`: Deliver the application (once build) using a basic Python HTTP Server.
- `setup`: Build and upload the application.
- `upload`: Upload the application (must be already build).

Please do this:
```
- import_role:
    name: javascript-app
  vars:
    jsapp_role_action: setup
```

Or this:
```
- hosts: localhost
  roles:
    - javascript-app
  vars:
    jsapp_role_action: setup
```

And not this:
```
- import_role:
    name: javascript-app
    tasks_file: setup.yml
```

### Dependencies

#### Variable `jsapp_python_packages`

Define Python packages required to install [nodeenv](https://github.com/ekalinin/nodeenv) using pip (nodeenv itself by default).

#### Variable `jsapp_python_version`

Define Python version to use for installing [nodeenv](https://github.com/ekalinin/nodeenv) (inside a virtual environment).

#### Variable `jsapp_pip_environment`

Defaults to the value of `python_pip_environment`.

#### Variable `jsapp_pip_umask`

Defaults to the value of `jsapp_pip_umask`.

### Build Parameters

#### Variable `jsapp_directory`

Where to checkout the application soure code during the build.

#### Variable `jsapp_production_mode`

Set production mode (defaults to `yes`).

### Required If Cloning From Git

Required only if `jsapp_repository_url` is an URL.

#### Variable `jsapp_repository_host`

The Git server hostname (e.g. `gitlab.your-corp.com`).

#### Variable `jsapp_repository_key`

The Git repository's SSH key (public key).

You can easily retrieve it using ssh-keyscan [link](https://unix.stackexchange.com/a/126911):

```
$ ssh-keyscan gitlab.your-corp.com
gitlab.your-corp.com: Connection closed by remote host
# gitlab.your-corp.com:22 SSH-2.0-OpenSSH_7.4
gitlab.your-corp.com ssh-rsa AA(some-more-characters-here)16v
# gitlab.your-corp.com:22 SSH-2.0-OpenSSH_7.4
gitlab.your-corp.com ecdsa-sha2-nistp256 (etc)
```

Set `jsapp_repository_key` to `gitlab.your-corp.com ssh-rsa AAA(some-more-characters-here)16v`.

#### Variable `jsapp_ssh_key_private`

Path to the private key used to clone the application source code (e.g. GitLab deploy key).

#### Variable `jsapp_ssh_key_public`

Path to the public key used to clone the application source code (e.g. GitLab deploy key).

#### Variable `jsapp_version`

Which version (commit, branch, tag) to checkout (only relevant if cloning from a Git repository).

### Required If Hosting On AWS S3

Required only if `jsapp_hosting_implementation` is set to `s3`.

#### Variable `jsapp_s3_bucket_name`

TODO

#### Variable `jsapp_s3_region`

TODO

#### Variable `jsapp_s3_permission`

TODO

## License

See [LICENSE.rst](LICENSE.rst).

## Authors

See [AUTHORS](AUTHORS).

2014-2019 - David Fischer

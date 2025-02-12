# SignalFx Agent Ansible Role

This is the `signalfx-agent` Ansible role that will install and configure the
SignalFx Smart Agent on your remote hosts.  To install it, you can clone the
agent repo to your controller and add the `signalfx-agent` role directory to
its `roles_path` in your `ansible.cfg`, or use this document's directory as
your working directory.  The role is also available via Ansible Galaxy:

```
ansible-galaxy install signalfx.smart_agent
```

To use this role, simply include the `signalfx-agent` role invocation in your
playbook (or `signalfx.smart_agent` if installed via Galaxy).  An
ansible-playbook call using the provided example playbook and variable files
would be similar to:

```
ansible-playbook -i your_inventory_file -e @example-config.yml example-playbook.yml
```

This role sources the following variables:

 - `sfx_agent_config`: A mapping that gets converted to the Smart Agent
   configuration YAML file. See the [Agent Config
   Schema](https://github.com/signalfx/signalfx-agent/blob/main/docs/config-schema.md)
   for a full list of acceptable options and their default values.  The only
   required key-value pair is `signalFxAccessToken`. 

    Here is a basic `sfx_agent_config` that will monitor a basic set of host-level components:

    ```yaml
    sfx_agent_config:
      signalFxAccessToken: MY-TOKEN  # Required
      monitors:
        - type: cpu
        - type: filesystems
        - type: disk-io
        - type: net-io
        - type: load
        - type: memory
        - type: vmem
        - type: host-metadata
        - type: processlist
    ```

	It is suggested to keep this mapping in a variable file in the respective
	`group_vars` or `host_vars` directory for your target remote hosts, or to
	pass it in via the `-e @path/to/variable_file` ansible-playbook extra vars
	option for a "global" configuration.

 - `sfx_conf_file_path`: The target path for the Smart Agent configuration
   file generated from `sfx_agent_config` (**default:**
   '/etc/signalfx/agent.yaml'). See `sfx_win_conf_file_path` for Windows platforms.

 - `sfx_repo_base_url`: The url provided to yum/apt for obtaining the SignalFx Smart Agent
   (**default:** `https://splunk.jfrog.io/splunk`)

 - `sfx_package_stage`: The package repo stage to use: `release`, `beta`, or `test`
   (**default:** 'release')

 - `sfx_version`: The agent package version.  This is of the form `<agent
   version>-<package revision>` (e.g. package version `3.0.1-1` is the first
   package revision that contains the agent version `3.0.1`).  Releases with
   package revision > 1 contain changes to some aspect of the packaging scripts
   (e.g. init scripts) but contain the same agent bundle. (**default:**
   'latest')

 - `sfx_service_user` and `sfx_service_group`: Set the user/group ownership for the
   signalfx-agent service. The user/group will be created if they do not exist.
   Requires agent package version 5.1.0 or newer. (**default:** 'signalfx-agent'). Not used
   in Windows platforms.

 - `sfx_win_install_path`: The Windows installation path. (**default:** 
   'C:\\Program Files\\SignalFxAgent')

 - `sfx_win_conf_file_path`: The Windows agent configuration file path. (**default:** 
   '\\ProgramData\\SignalFxAgent\\agent.yaml')

 - `sfx_win_dl_base_url`: The base URL for Windows agent downloading (**default:** 
   'https://dl.signalfx.com')

 - `sfx_skip_repo` (Linux only): If installing the agent from a custom or self-hosted
   apt/yum repo, set to `true` to skip the installation of the default repo
   (**default:** `false`)

**Note**: On Linux, after the `signalfx-agent` role is deployed, Ansible will manage the
`signalfx-agent` service via the Ansible core `service` module.  This module
will automatically determine the host's init system for starting/stopping the
`signalfx-agent` service, with a preference for systemd (`systemctl`).

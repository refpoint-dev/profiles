# refpoint profiles

[![License: MIT](https://img.shields.io/badge/license-MIT-22a5d6)](LICENSE)

Ready-to-use compliance and hardening profiles for
[refpoint](https://github.com/refpoint-dev/refpoint), the tool that checks
machines are in the state you want them in and tells you when they are not.

Each directory is one profile: a `profile.rpp` manifest with its metadata,
variables and allowed commands, plus a `controls/` folder of `.rpc` files
holding the rules.

## Profiles

| Profile | Controls | What it checks |
|---|---:|---|
| [`linux-baseline`](linux-baseline) | 64 | Distribution-independent Linux server hardening: partitioning and mount options, automatic updates, SSH, accounts and passwords, sudo, auditd, logging, time sync, MAC, kernel parameters, services, cron and banners. |
| [`linux-firewall`](linux-firewall) | 46 | Host firewall posture with ufw, firewalld or nftables. Backend-specific controls are gated on the backend being installed; shared controls cover the effective policy, kernel network hardening, listening services and optional fail2ban. |
| [`docker`](docker) | 41 | Docker Engine host hardening: `daemon.json`, service state, socket and file permissions, audit rules, kernel parameters and running-container posture. |
| [`harbor`](harbor) | 56 | Harbor registry hardening for installs made with the official installer: configuration, TLS, authentication, network exposure, container runtime, data protection, logging and API checks across every component. |
| [`harbor-health`](harbor-health) | 14 | Lightweight Harbor operational check: every component container is running the expected `goharbor` release and the health API reports all components healthy. |
| [`example`](example) | 2 | A minimal profile showing profile, control and check syntax. A good starting point for writing your own. |

All profiles except `example` target Linux.

## Usage

Install refpoint from its
[releases page](https://github.com/refpoint-dev/refpoint/releases), clone this
repository, then run a profile:

```shell
git clone https://github.com/refpoint-dev/github-profiles.git
cd github-profiles

# Validate a profile and list the executables it needs, without touching a target
refpoint check linux-baseline

# Run against the local machine
refpoint run linux-baseline

# Run against a remote host over SSH
refpoint run linux-baseline -t admin@web-01
```

Point `run` at a directory of profiles to run them all.

### Allowed commands

Controls that use the `command` resource only run executables that both the
profile and the operator permit. Each profile declares what it needs in
`allowed_commands`; you set the ceiling with `--allowed-commands`. Use
`refpoint check <profile>` to see the list a profile requires:

```shell
refpoint run docker --allowed-commands docker,xargs,grep,printf,sort,head,test,find
```

### Variables

Every path, port, user and threshold is a variable with a sensible default,
so you can adapt a profile to your site policy without editing its controls.
The variables and their descriptions are declared in each `profile.rpp`.

Override them on the command line or from an HCL file:

```shell
refpoint run harbor-health --var harbor_version=v2.15.0 --var base_url=https://harbor.example.com
refpoint run harbor-health --var-file my-vars.rpv
```

[`harbor-health/vars.rpv`](harbor-health/vars.rpv) is an example variable file.

### Reports

```shell
refpoint run linux-baseline --format json --output run.json
refpoint run linux-baseline --reporter junit:report.xml --reporter json:run.json
```

See the [refpoint documentation](https://github.com/refpoint-dev/refpoint#readme)
for every output format, targets file, and option.

## Profile layout

```
<profile>/
├── profile.rpp        # name, title, version, platform, variables, allowed_commands
├── controls/
│   └── *.rpc          # control blocks, each with one or more checks
└── vars.rpv           # optional variable overrides
```

A control looks like this:

```hcl
control "example-01" {
  title  = "Hosts file is present"
  impact = 0.5
  tags = {
    category = "example"
  }

  check "hosts-file" "file" {
    path   = "/etc/hosts"
    exists = true
  }
}
```

## Contributing

Issues and merge requests are welcome. When adding or changing a profile:

- Keep site-specific values in variables, not in controls.
- Declare every executable a `command` check runs in `allowed_commands`,
  including those after pipes and in `$( )` substitutions.
- Gate controls on the component being present so they skip, rather than fail,
  on hosts where they do not apply.
- Run `refpoint check <profile>` before submitting.

## License

The profiles in this repository are licensed under the [MIT License](LICENSE).
refpoint itself is licensed separately under AGPL-3.0-only.

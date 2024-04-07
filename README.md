**STATUS: EARLY ALPHA***

This repo contains a "VVV" Vagrant / VirtualBox based local development environment setup for jkl.hacklab.fi site.

It supports GIT based theme and plugin development workflow using popular PHP/WP development tools:

- PHP error logs at `www/hacklab-jkl/wp-content/debug.log`
- [WP-CLI](https://make.wordpress.org/cli/handbook/guides/quick-start/) tool
- PhpMyAdmin and readiness to [connect external SQL clients](https://varyingvagrantvagrants.org/docs/en-US/references/sql-client/) like DBeaver etc.
- IDE features
  - LSP (go-to-declaration etc. just start editor in www-root), 
  - XDebug (see notes on later section for config)
  - Profiling tools, mail diagnostics, etc...
- Automatic hosts file config for development domain, access via http://jkl.hacklab.test
- Composer etc.

It works on computers with X86/AMD64 CPU. Apple Silicon Mac users would need to use commercial version of Parallels to be able to run this (YMMV) or use something else like [Docker based WP-ENV](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/) or [LocalWP app](https://localwp.com/). The basic sync script `pull-production.sh` written in bash could also be useful as a starting point with other development environment setups.

## Setup with existing VVV environment
Add to config.yml:

    hacklab-jkl:
    skip_provisioning: false
    description: "Hacklab Jyväskylä"
    repo: https://github.com/Varying-Vagrant-Vagrants/custom-site-template.git
    php: 8.2
    hosts:
      - jkl.hacklab.test
    custom:
      wpconfig_constants:
        WP_DEBUG: true
        WP_DEBUG_LOG: true  # logs to wp-content/debug.log

Then run `vagrant reload --provision` and site should be initialized in few minutes. Skip over the next "Setup from scratch" section.

## Setup from scratch

By default the VVV stock configuration is set to create http://jkl.hacklab.test site at `www/hacklab-jkl/public_html`. Additionally http://one.wordpress.test and http://two.wordpress.test stock WP sites are available for testing.

Clone repo to a drive with few gigabytes of free space and provision Vagrant (first run takes ~15 minutes):

```
git clone git@github.com:HacklabJKL/VVV.git
cd VVV
vagrant plugin install --local
vagrant up --provision
```

## Pulling live site to development environment
After finishing initial setup, make sure you have working server credentials with public key authentication set up to Hacklab Jkl WordPress server "Jonne" to be able to pull production files and database to the local development environment.

Add "jonne" into local development machine's SSH config eg. with `vi ~/.ssh/config`:

```
Host jonne
hostname jkl.hacklab.fi
user <ssh_login_user_name_here>
port 22
```

Validate that `ssh jonne` commands connects to server correctly and then then run sync script to pull files and database from production site:

```
./pull-production.sh
```

After which http://jkl.hacklab.test site should be available for logging in with production credentials.

## HTTPS setup

By default the local site is set up to use HTTP instead of HTTPS, which should be fine for most local development tasks and installs with least problems.

VVV supports HTTPS however and to enable it, refer to https://varyingvagrantvagrants.org/docs/en-US/references/https/ on how to setup CA and accept self-signed certs in browser.

Then comment out following line in the end of `./pull-production.sh` and run script again to keep https-urls intact.

```
vagrant ssh -c "wp --path=/srv/www/hacklab-jkl/public_html search-replace \"https://\" \"http://\" --dry-run"  # comment this to use https
```

## Notes about pushing changes to production

Changes in themes or plugins should be versioned in separate GIT repositories, pushed to remote (eg. Github) and then pulled on production server.

This should work fine for low volume collaboration as a starting point. 

Local GIT repo state will be overridden when running `pull-production.sh`, so it's best to keep it clean.

Pushing changes that are saved in database or filesystem (eg. page content, customizer settings, widgets, wp settings etc.) from local to production is not supported. It is theoretically possible however.

## Note about uploads folder

All server pictures and media files are downloaded to local development environment, which can be turned off in script with `rsync --exclude 'uploads'` that is commented out as example.

After that all upload folder pictures will give 404's which can be annoying. Nginx can be set to provide placeholder jpg's in place of actual images via some url rewrite config trickstery.

## XDebug step debugger

Map `/home/<local_username>/<path_to>/vvv/www/` path to `/srv/www/` and it should work. Run `xdebug_on` command in Vagrant if it doesn't enable XDebug for some reason (eg. `vagrant ssh -c "xdebug_on"`).

Example NeoVim config:

```
  local dap = require("dap")

  -- Config for a VVV (Vagrant) WordPress site
  dap.configurations.php = {
    {
      type = "php",
      request = "launch",
      name = "Listen for VVV Xdebug",
      port = 9003,
      localSourceRoot = "/home/user/sites/vvv/www/",
      serverSourceRoot = "/srv/www/",
    },
  }
```

Refer to [VVV XDebug docs](https://varyingvagrantvagrants.org/docs/en-US/references/xdebug/) for more examples etc.

## Usage with existing VVV setup etc.

WordPress www-root can be also symlinked as `./public_html` to same folder as `pull-production.sh` script, and it will use it if it's found. This way the script can be used independently, also with other setups. 

It was simply embedded with this VVV repository as convenience for someone who has not ran WordPress before and is interested in developing Hacklab Jkl WordPress web site with least hassle.


Original VVV repository readme continues...

# VVV

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/206b06167aaf48aab24422cd417e8afa)](https://www.codacy.com/gh/Varying-Vagrant-Vagrants/VVV?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Varying-Vagrant-Vagrants/VVV&amp;utm_campaign=Badge_Grade) [![Average time to resolve an issue](http://isitmaintained.com/badge/resolution/varying-vagrant-vagrants/vvv.svg)](http://isitmaintained.com/project/varying-vagrant-vagrants/vvv "Average time to resolve an issue") [![Percentage of issues still open](http://isitmaintained.com/badge/open/varying-vagrant-vagrants/vvv.svg)](http://isitmaintained.com/project/varying-vagrant-vagrants/vvv "Percentage of issues still open")

VVV is a local developer environment, mainly aimed at [WordPress](https://wordpress.org) developers. It uses [Vagrant](https://www.vagrantup.com) and VirtualBox/Parallels/HyperV to create a linux server environment for building sites, and contributing to WordPress itself.

_VVV stands for Varying Vagrant Vagrants._

## How To Use

To use it, download and install [Vagrant](https://www.vagrantup.com) and a provider such as [VirtualBox](https://www.virtualbox.org/), Docker, or Parallels Pro. Then, clone this repository and run:

```shell
vagrant plugin install --local
vagrant up --provision
```

When it's done, visit [http://vvv.test](http://vvv.test).

The online documentation contains more detailed [installation instructions](https://varyingvagrantvagrants.org/docs/en-US/installation/).

* **Web**: [https://varyingvagrantvagrants.org/](https://varyingvagrantvagrants.org/)
* **Contributing**: Contributions are more than welcome. Please see our current [contributing guidelines](https://varyingvagrantvagrants.org/docs/en-US/contributing/). Thanks!

## Minimum System requirements

[For system requirements, please read the system requirements documentation here](https://varyingvagrantvagrants.org/docs/en-US/installation/software-requirements/)

## Software included

For a comprehensive list, please see the [list of installed packages](https://varyingvagrantvagrants.org/docs/en-US/installed-packages/).

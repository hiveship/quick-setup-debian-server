# Quick Setup Debian Server

Ansible setup to configure a blank Debian 8 server automatically with only one command.

# Quick Start

1. Customise variables in `./group_vars/all/vars_file.txt` and change extention of the file from `.txt` to `.yml`.
2. Run `make init` for first run of the script (on a blank server) or `make update` to update an already secured server.

# Included in this Setup

* Securisation of the server:
    - Create a new user with its own password,
    - Disable root & password authentification (use of SSH keys),
    - Install and configure security services, including:
        + Fail2ban
        + Rkhunter
    - Change SSH default port.
* Installation of Newrelic agent to monitor the server.
* Installation of basic stuff for various uses:
    - `htop`,
    - `Python`,
    - ...
* Installation of Docker. This will be the only thing running in the server considered, all services will be dockerized.
* Installation of various Docker services:
    - `reverse-proxy`: install & configure a reverse proxy to route all the http requests to the good dockerized service. Based on `jwilder/nginx-proxy` docker scripts. See: https://github.com/jwilder/nginx-proxy
    - `srv-voeux`: install & configure a php website used in my portfolio. _You probably won't need that... ;)_
    - `seedbox`: install & configure several seedbox. Based on `kfei/docktorrent` docker scripts. See: https://github.com/kfei/docktorrent
    - `files`: install & configure several https web interface to retrieve the files downloaded by the seedbox. Based on `smdion/docker-h5ai` docker scripts. See: https://hub.docker.com/r/smdion/docker-h5ai/.
    - `jenkins`: install & configure Jenkins.
    - `gogs`: install & configure a git repo for this server. Use of gogs, see: https://github.com/gogits/gogs
    - `cadvisor`: install & configure cAdvisor to monitor all the services created. Use of cAdvisor, see: https://github.com/google/cadvisor.
    - `owncloud`: install & configure an ownCloud server.

# How to Run the Scripts

1. Identify which services you need. The file `main.yml` is used to tell Ansible which roles to play. Simply comment/uncomment the ones you don't need. :)
2. Customise variables. Under `./group_vars/all/`, there is a file named `vars_file.txt`:
  * Change all the variables you need inside of this file,
  * Change the extension of the file from `.txt` to `.yml`.
3. Create template files (SSL certs, htpasswd files, ...) needed to configure the different services (see next section for more explanations about the templates and files needed)
4. Run the command `make (init|update)` (init for blank server, update for already secured server); type the password you have configured in `./group_vars/all/vars_file.txt`
5. Grab a coffee, chill out... Because everything is done now - you only have to wait for the end of the execution. :)

_Note: if for some reason, the deployment fails after the securisation of the server, you should comment the `- securisation` role. If you forget to do so, you will have an error (server unreachable) because root auth is disabled & SSH port is changed after the securisation!_

# Specific Configurations
## Main Configuration File
### Unix Encrypted Password
To generate the encrypted password of your user:

```
mkpasswd --method=sha-512
```

If this utility is not installed on your system then you can still easily generate these passwords using Python. See: http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module

### Seedbox

It is possible to run multiple seedbox on the same server. If you want to do so, a `rtorrent` will be used as a master and this master can have several slaves.

That is why there is two Ansible variable dictionaries: one for `seedbox_master` and one for `seedbox_slave`. 

* `ports` are the ports you need to expose to make the service run 
* `name[s]` are the names of the docker containers.

Further information concerning the customisation of the seedboxes can be found at the following address, see: https://github.com/kfei/docktorrent

### Hostnames

If you are using a reverse proxy, it is certainly because you have `some-url.pointing-to-your-server.co` ; in this section, you can customise the URL of each of the services.

## Reverse-proxy
### "tasks/my_specialconfs_pub.yml"

Configure certs, special configurations and htpassword for desired services. For more details, see:

* SSL: https://github.com/jwilder/nginx-proxy#ssl-support
* HTTP Auth: https://github.com/jwilder/nginx-proxy#basic-authentication-support
* Special configuration for one service: https://github.com/jwilder/nginx-proxy#per-virtual_host

### "templates/"

This folder contains everything needed in previous step, such as:
- Certs,
- htpasswd files,
- Special configuration files...

## Seedbox
### "tasks/my_specialconfs_pub.yml"

Copy htpasswd files for seedboxes.

### "templates/"

This folder contains the htpasswd files needed in previous step.

# Improvements & TODOs

- Bugs:
    * `mariadb` for owncloud on some servers,
    * `python-pip` installation fails sometime.
- Refactor code (in particular variables structure) to have a better code quality,
- Improve workaround for having multiple seedboxes on the same host: using screens is not a brilliant solution, and I am really interested in something prettier (maybe customizing the `Dockerfile` of rtorrent?),
- Write tests.

If you find bugs & improvements, feel free to submit an issue, fork this project and send pull requests! :) 




# Testilda

Testilda is our end-to-end automation framework. It is used to test the front end of both CA & CE products.

## Architecture
<img width="310" alt="Screen Shot 2021-04-07 at 12 43 56" src="https://user-images.githubusercontent.com/14069474/113854387-1bfb9980-979f-11eb-94d5-b31fb46c7e4d.png">

[RSpec info](https://rspec.info/)

[Capybara](https://github.com/teamcapybara/capybara)

[Web driver](https://www.selenium.dev/documentation/en/webdriver/#:~:text=Selenium%20WebDriver%20refers%20to%20both,and%20more%20concise%20programming%20interface.)



## Setup

### Cloning the repo

The Testilda repo contains a snapshot file, that is getting too large to be handled in normal GIT repo. Therefore we are now using GIT LFS.

First you need to install LFS support to GIT: https://github.com/git-lfs/git-lfs/wiki/Installation

Then run `git lfs install` and `git lfs pull` in the root of MAB. The snapshot will now be downloaded. Everything else will now be handled automatically.

### Product setup

When using a monostack you can set up the product on your monostack and run testilda locally, but make sure you point any links to your monostack (further on). 

Follow the docker/kubernetes workflow instructions for building.

**Update .env file for testilda.** 

An up to date example of an .env file working with testilda on last update is given in `testilda/.env.SAMPLE`.

```sh
COMPOSE_HTTP_TIMEOUT=500
MAB_CONFIG_PLATFORM_DOMAIN=***
MAB_CONFIG_ENCRYPTION_SECRET=testilda
MAB_CONFIG_ENCRYPTION_SALT=testilda
MAB_CONFIG_NUCLEAR_WEAPONS_ALLOWED=1
MAB_CONFIG_MOCK_APPLICATION=1
MAB_CONFIG_PLATFORM=uab
MAB_CONFIG_USE_CELTRA_AUTH=0
#MAB_CONFIG_DEV_PROXY_ENABLE_SSL=0  # for kubernetes monostack
```

**Import db fixtures**

Just before running the admin-task upgrade command you need to import the Snowflake (analytics) and Mysql (operational) fixtures. Run the following commands when using:

Docker:
```sh
workflows/docker/bin/snapshot restore ./testilda/fixtures/testilda.mabsnapshot
docker-compose run --rm admin-task upgrade  # this is now run automatically by workflows/docker/bin/snapshot, but might be useful
```

Kubernetes:
```sh
workflows/kubernetes/bin/snapshot restore ./testilda/fixtures/testilda.mabsnapshot
k task upgrade
```

You don't need to always run all services. You can look for the relevant services in the `testilda/testilda_services` file.

### Local file prep

1. In `testilda` folder:

    Copy `config.SAMPLE.yml` into `config.yml` and change hosts (`host_name` and `hub_host_relative`) to use your platform domain. For example `host_name: mab.<your name>.test/`. Leave the `/` at the end. 
    
    When running the product **remotely** (monostack) leave `hub_host_relative` in `config.yml` unchanged - as it was in `config.SAMPLE.yml`. 

1. When running the product locally, change your `/etc/hosts` file to include testilda accounts listed in your `config.yml` file. Example of line added to hosts file for `autotest` account:
    ```sh
    <your ip> autotest.<host_name>
    ...
    ```

### Create and connect to testilda

First build image with `docker-compose build` from `testilda` subfolder.

Then run it with `docker-compose up` from the same subfolder.

If you are using a kubernetes monostack for testilda, you also have to increase the allocated memory of the container:
`docker update --memory 3g --memory-swap 4g [TestildaContainerID]`

* Running tests flat:
  * Run `testilda/bin/access-container` and run tests with rspec
* Using VNC to view browser:
  * Connect with VNC viewer to localhost:5900, pass: abacadae
  * Run `testilda/bin/access-container` and run tests with rspec
  * Or use xterm in VNC viewer to run tests with rspec
  * If you are using testilda on Monostack, connect to it with forwarding port 5900: `ssh instanceName -L 5900:localhost:5900` and then use same `localhost:5900` with VNC viewer or Screen share to connect

The `/testilda` folder is mapped to your local testilda copy.

## Running and writing tests

### Running tests with rspec

To run a test file run
```sh
rspec spec/01_login/login_spec.rb
```

or run a specific test on line whose it is on line 10:
```sh
rspec spec/01_login/login_spec.rb:10
```

### Linting
Before committing your code, run linter outside of testilda container with
```sh
testilda/bin/lint -a
```

If you add `-a` at the end, linter will do some smaller automatic fixes, that should not cause any problems.

If you add `-A` (capital) in the end, linter will force automatic fixes, which can break some things sometimes.

### Upgrading fixtures

First follow the import procedure and upgrade Snowflake and DB to latest version. 
Make sure there aren't any undesired changes in the database. 

MAB ONLY: Make sure that the custom audience tables are not present. If the audiences are present you need to first:
```sh
docker-compose run --rm admin-task exec bash
services/custom-audiences-background/bin/truncate-custom-audiences
```

To create a snapshot of the your current db state when using docker run:
```sh
workflows/docker/bin/snapshot -n testilda create path/to/desired/folder/
```
or using kubernetes:
```sh
workflows/kubernetes/bin/snapshot -n testilda create path/to/desired/folder/
``` 
Make sure that snapshot has write permission to your chosen folder otherwise creating of snapshot will fail.
After snapshot is created MAKE SURE TO CHECK IT'S SIZE. It should not differ too much from the one used as base.

## Debugging

put `binding.pry` in code for breakpoint

To open pry session automatically when something fails, run `sudo gem install pry-rescue pry-stack_explorer` inside container and run tests with `rescue rspec spec/...`

### Debug on circle

[How to SSH to circle](https://docs.google.com/document/d/1boYX7-1nrwgfAQ13dadeiVibc4tQfIigmKFTV2jIP50/edit?pli=1#heading=h.y19e7x9vbaqy)


## [wip, uab, [#644](https://github.com/celtra/uab/pull/644)] Create and connect to testilda (Kubernetes with Helm)

First build image with `docker-compose build` from `testilda` subfolder.

Then start it with `k up testilda` from root git folder.

* Running tests flat:
  * Run `k exec testilda bash` and exec `cd ./testilda` to change folder, then run tests with rspec
* Using VNC to view browser:
  * Proxy VNC port:
    `kubectl port-forward $(kubectl get pods | grep testilda | awk '{print $1}') 5900:5900`
  * Connect with VNC viewer to localhost:5900, pass: abacadae
  * Press enter and run tests with rspec
  * Or use xterm in VNC viewer to run tests with rspec

The `/testilda` and `/shared` folders are mapped to your local testilda copy.

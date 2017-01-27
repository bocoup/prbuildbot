# PR Build Bot
The PR Build Bot is a Python web application that listens to Travis CI webhooks
and posts selections from the build logs as comments on the GitHub PR. This
allows commenting on PRs from both trusted and untrusted branches of the main
repository without exposing the GitHub Personal Access Token of the commenting
user.


## Installation on Dedicated Server

This installation process uses [Ansible](http://docs.ansible.com/) to configure
the server to use [nginx](http://nginx.org/) and
[uWSGI](http://uwsgi-docs.readthedocs.io/en/latest/) to serve a
[Flask](http://flask.pocoo.org/) application. It expects nginx to be able to
use port 80.

### Requirements

- Ubuntu-like server environment
- [git](https://git-scm.com/downloads)
- [Ansible >= 2.2.1.0](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip)
  - Note: on Ubuntu, this requires installing ansible via `pip`, not `apt`.

### Process

This is how to install the dependencies and application on a base Ubuntu 16.04
box. Git should come out of the box.

1. Create a server instance wherever you like.
2. Add a non-root user to own the application.
  - `adduser prbuildbot` as root
  - You will create a password for this user. Keep it secret. Keep it safe.
3. Ensure the new user is able to `sudo`.
  - `adduser prbuildbot sudo` as root
4. Log into the server as the new user.
5. Update apt repositories
  - `sudo apt-get update`
6. Install pip.
    `sudo apt install python-pip`
7. Upgrade pip.
  - `sudo -H pip install --upgrade pip`
8. Install libssl-dev.
  - `sudo apt install libssl-dev`
9. Install Ansible.
  - `sudo -H pip install ansible`
10. Fork this repository and clone it into the user's home directory.
  - `git clone https://github.com/<your user name>/prbuildbot.git`
  - You will be editing a file under version control, so you should create your
    own fork and store your changes there.
11. Configure the application for your project. See
    [Configuration](#configuration), below.
12. Change into the `ansible` directory and run the provisioning script.
  - `cd prbuildbot/ansible`
  - `ansible-playbook provision.yml`
  - Note: this must be run from the `prbuildbot/ansible` directory, or it
    will fail.
  - **Note:** This does not install any of the python modules into virtual_env,
    but at the system level. This is possible with Ansible, just not implemented
    yet.


## Configuration

You will need to set up a user on GitHub, get a personal access token for them,
and set up the configuration file for the application.

1. Set up a user on GitHub that you want to be the "commenter."
2. Get a Personal Access Token for that user.
  - Go to https://github.com/settings/tokens.
  - Click "Generate new token"
  - On the creation page, name the token and give it at least `public_repo`
    and `user:email` permissions.
3. Set up the configuration file on the server.
  - In the application directory: `cp config.sample.txt config.txt`
  - Edit the config properties as necessary (see below for descriptions)
4. Edit `log_parser.py` to parse the Travis CI job logs in whatever way you
   require.
  - The included `log_parser.py` includes the logic for parsing log files for
    `w3c/web-platform-tests` as an example.
  - You should commit this file back into your fork so that you don't lose it
    in case you lose or change your server.
  - Do not ever commit your config.txt, as it contains your Personal Access
    Token.
5. Add a webhook notification in your main project's `.travis.yml` file:
  - ```
    notifications:
      webhooks: http://<your-server-here>/prbuildbot/travis
    ```
  - Your server can be referenced by either an IP address or a fully-qualified
    domain name.
6. Once your updated `.travis.yml` file is in your project, you should start
   receiving comments on pull requests from this bot.
   - Example: https://github.com/bobholt/web-platform-tests/pull/4

### Configuration Properties

| Property        | Description                                                                                                                                         |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| TRAVIS_URL      | The Travis CI api endpoint that applies to your application (either .com or .org)                                                                   |
| COMMENT_ENV_VAR | An environment variable used in your Travis CI build matrix that serves as a flag for whether or not the job's log should be parsed as a PR comment |
| GH_TOKEN        | The Personal Access Token created in Configuration Step 2, above                                                                                    |
| ORG             | The GitHub organization/owner of the main repository (this project's ORG would be "bobholt")                                                        |
| REPO            | The main repository (this project's REPO would be "prbuildbot")                                                                                     |

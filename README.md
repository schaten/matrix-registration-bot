# Matrix Registration Bot

![Pypi badge](https://img.shields.io/pypi/v/matrix-registration-bot.svg)
![License](https://img.shields.io/pypi/l/matrix-registration-bot?color=%23008000)
![Docker pulls](https://img.shields.io/docker/pulls/moanos/matrix-registration-bot)

This bot aims to create and manage registration tokens for a matrix server. It wants to help invitation based servers to
maintain usability. It does not create a user itself, but allows registration only with a valid token as defined by
Matrix standard
[MSC3231](https://github.com/matrix-org/matrix-doc/blob/main/proposals/3231-token-authenticated-registration.md). The
benefit is, that an administrator minimizes manual work and does not know a user's password at any time.

This means, that a user that registers on your server has to provide a registration token to successfully create an
account. The token can be created by interacting with this bot. So to invite a friend you would send `create` to the bot
which answers with a token. You send the token to the friend, and they can use this to create an account.

The feature was added in Matrix v1.2. More information can be found in the
[Synapse Documentation](https://matrix-org.github.io/synapse/latest/usage/administration/admin_api/registration_tokens.html)
.

If you have any questions, or if you need help setting it up, read the [troublshooting guide](./docs/troubleshooting.md)
or join [#matrix-registration-bot:hyteck.de](https://matrix.to/#/#matrix-registration-bot:hyteck.de).

# Supported commands

**Unrestricted commands**

* `help`: Shows this help

**Restricted commands**

* `list`: Lists all registration tokens
* `show <token>`: Shows token details in human-readable format
* `create`: Creates a token that that is valid for one registration for seven days
* `delete <token>` Deletes the specified token(s)
* `delete-all` Deletes all tokens
* `allow @user:example.com` Allows the specified user (or a user matching a regex pattern) to use restricted commands
* `disallow @user:example.com` Stops a specified user (or a user matching a regex pattern) from using restricted
  commands

# Permissions

By default, any user on the homeserver of the bot is allowed to use restricted commands. You can change that, by using
the `allow` command to configure one (or multiple) specific user. Read
the [simple-matrix-bot documentation](https://simple-matrix-bot-lib.readthedocs.io/en/latest/manual.html#allowlist)
for more information. If you get locked out for any reason, simply modify the config.toml that is created in the bots
working directory.

# Getting started

## Install via [matrix-docker-ansible-deploy](https://github.com/spantaleev/matrix-docker-ansible-deploy)

If you already installed your homeserver with this ansible playbook you can make use of a very simple setup. Check out [the setup instructions in the project's repo](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook-bot-matrix-registration-bot.md).

## Prerequisites for all other installation methods

### Server configuration

Your server should be configured to a token restricted registration. Add the following to your `homeserver.yaml`:

```yaml
enable_registration: true
registration_requires_token: true
```

## Create a bot account

Then you need to create an account for the bot on the server, like you would do with any other account. A good username
is `registration-bot`. If you want to use token based login, note the access token of the bot. One way to get the token
is to log in as the bot and got to `Settings -> Help & About -> Access Token` in Element, however you mustn't log out or
the token will be invalidated. As an alternative you can use the command

```shell
curl -X POST --header 'Content-Type: application/json' -d '{
    "identifier": { "type": "m.id.user", "user": "YourBotUsername" },
    "password": "YourBotPassword",
    "type": "m.login.password"
}' 'https://matrix.YOURDOMAIN/_matrix/client/r0/login'
```

Once you are finished you can start the installation of the bot.

## Manual Installation

The installation can easily be done via [PyPi](https://pypi.org/project/matrix-registration-bot/)

```bash
$ pip install matrix-registration-bot
```

## Configuration

Configure the bot with a file named `config.yml`. It should look like this

```yaml
bot:
  server: "https://synapse.example.com"
  username: "registration-bot"
  access_token: "verysecret"
  # It is also possible to use a password based login by commenting out the access token line and adjusting the line below
  # password: "secretpassword"
  prefix: ""
api:
  # API endpoint of the registration tokens
  base_url: 'https://synapse.example.com'
  # Access token of an administrator on the server. If you configured the bot to be an admin on the sever you can use the same token as above.
  token: "supersecret"
logging:
  level: DEBUG/INFO/ERROR
```

It is also possible to use environment variables to configure the bot. The variable names are all upper case,
concatenated with `_` e.g. `LOGGING_LEVEL`.


### Start the bot

Start the bot with

```bash
python -m matrix_registration_bot.bot
```

and then open a Direct Message to the bot. The type one of the following commands.

### Automatically (re-)start the bot with Systemd

To have the bot start automatically after reboots create the file `/etc/systemd/system/matrix-registration-bot.service`
with the following content on your server. This assumes you use that you place your configuration in
`/matrix/matrix-registration-bot/config.yml`.

```
[Unit]
Description=matrix-registration-bot

[Service]
Type=simple

WorkingDirectory=/matrix/matrix-registration-bot
ExecStart=python3 -m matrix_registration_bot.bot

Restart=always
RestartSec=30
SyslogIdentifier=matrix-registration-bot

[Install]
WantedBy=multi-user.target
```

After creating the service reload your daemon and start+enable the service.

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start matrix-registration-bot
$ sudo systemclt enable matrix-registration-bot
```

## Install using docker-compose

To use this container via docker you can create the following `docker-compose.yml` and start the container
with `docker-compose up -d`. Explanation on how to obtain the correct values of the configuration can be found in the 
**Manual installation** section.

``` yaml
version: "3.7"

services:
  matrix-registration-bot:
    image: moanos/matrix-registration-bot:latest
    environment:
      LOGGING_LEVEL: DEBUG
      BOT_SERVER: "https://synapse.example.com"
      BOT_USERNAME: "registration-bot"
      BOT_PASSWORD: "password"
      API_BASE_URL: 'https://synapse.example.com'
      API_TOKEN: "syt_xxxxxxxxxxxxxxxxxxxxxxxx"

```

# Contributing

Feel free to contribute or discuss this bot
at [#matrix-registration-bot:hyteck.de](https://matrix.to/#/#matrix-registration-bot:hyteck.de)
or simply open issues and PRs here.

[Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/)

# Related Projects

* The project is made possible by [Simple-Matrix-Bot-Lib](https://simple-matrix-bot-lib.readthedocs.io).
* An alternative for managing tokens is [Synapse Admin](https://github.com/Awesome-Technologies/synapse-admin)



# TalkEQ

[![GoDoc](https://godoc.org/github.com/xackery/talkeq?status.svg)](https://godoc.org/github.com/xackery/talkeq) [![Go Report Card](https://goreportcard.com/badge/github.com/xackery/talkeq)](https://goreportcard.com/report/github.com/xackery/talkeq)

[![Total alerts](https://img.shields.io/lgtm/alerts/g/xackery/talkeq.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/xackery/talkeq/alerts/)

[![Platform Tests & Build](https://github.com/xackery/talkeq/actions/workflows/build_workflow.yml/badge.svg?branch=master)](https://github.com/xackery/talkeq/actions/workflows/build_workflow.yml)

TalkEQ bridges links between everquest and other services. Extends [DiscordEQ](https://github.com/xackery/discordeq).

## Setup

* Go to [releases](https://github.com/xackery/talkeq/releases) and download the latest exe or binary for your operating systsem.
* Go to https://discordapp.com/developers/ and sign in
* Click New Application the top right area
* Write anything you wish for the app name, click Create App
* Start the talkeq executable once. This generates a talkeq.conf file
* Copy the Application ID into your talkeq.conf's discord client_id section
* On the left pane, click Bot
* Press Add Bot, Yes, do it!
* Click the Reset Token button, Yes, do it!
* Press the copy button in the Token section
* Uncheck the Public Bot option
* Scroll to the bottom of the bot section, and toggle the Message Content Intent option ([Due to this fix](https://discord.com/developers/docs/change-log#sep-1-2022))
* Replace on this link's {CLIENT_ID} field with the client ID you obtained earlier. https://discordapp.com/oauth2/authorize?&client_id={CLIENT_ID}&scope=bot&permissions=268504064
* Open the link and authorize your bot to access your server.
* Ensure the bot now appears offline on your server's general channel

### Configure TalkEQ

* Start talkeq up. The first run, it will say `a new talkeq.conf file was created. Please open this file and configure talkeq, then run it again.`.
* Edit the talkeq.conf, walking through each section and applying it for your situation. There are comments that help you through the process.

### Configure discord users to talk from Discord to EQ

#### Using Discord Roles

* (Admin-level accounts on Discord can only do the following steps.)
* Inside discord go to Server Settings.
* Go to Roles.
* Create a new role, with the name: `IGN: <username>`. The `IGN:` prefix is required for DiscordEQ to detect a player and is used to identify the player in game, For example, to identify the discord user `Xackery` as `Shin`, Create a role named `IGN: Shin`, right click the user Xackery, and assign the role to them.
* If the above user chats inside the assigned channel, their message will appear in game as `Shin says from discord, 'Their Message Here'`

#### Using Users Database

* When talkeq runs, a users.txt file is generated the same directory as talkeq. Peek at the file to see the layout.
* If you write to this file, talkeq will hot reload the contents and update it's lookup table in memory for mapping users from discord to telnet (eq)
* You can write a website to edit this file, or by hand, to update talkeq and sync your player IGN tags

### Troubleshooting

- **I can talk from in game to discord, but messages in discord to in game fail with "message too small, ignoring, original message:"**: Double check the bot section, and toggle the Message Content Intent option. If this is disabled, the bot just sees empty content messages and fails.

/etc/init.d/talkeq
change APPDIR/APPBIN, user, and group to your set options
```sh
!/bin/sh

### BEGIN INIT INFO
# Provides:          talkeqdaemon
# Required-Start:    $local_fs $network $syslog
# Required-Stop:     $local_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: TalkEQ
# Description:       TalkEQ start-stop-daemon - Debian
### END INIT INFO

NAME="talkeq"
PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin"
APPDIR="/home/eqemu/talkeq/"
APPBIN="/home/eqemu/talkeq/talkeq"
APPARGS=""
USER="eqemu"
GROUP="eqemu"

# Include functions
set -e
. /lib/lsb/init-functions

start() {
  printf "Starting '$NAME'... "
  start-stop-daemon --start --chuid "$USER:$GROUP" --background --make-pidfile --pidfile /var/run/$NAME.pid --chdir "$APPDIR" --startas /bin/bash -- -c "exec $APPBIN > /var/log/talkeq.log 2>&1"
  printf "done\n"
}
#We need this function to ensure the whole process tree will be killed
killtree() {
    local _pid=$1
    local _sig=${2-TERM}
    for _child in $(ps -o pid --no-headers --ppid ${_pid}); do
        killtree ${_child} ${_sig}
    done
    kill -${_sig} ${_pid}
}

stop() {
  printf "Stopping '$NAME'... "
  [ -z `cat /var/run/$NAME.pid 2>/dev/null` ] || \
  while test -d /proc/$(cat /var/run/$NAME.pid); do
    killtree $(cat /var/run/$NAME.pid) 15
    sleep 0.5
  done
  [ -z `cat /var/run/$NAME.pid 2>/dev/null` ] || rm /var/run/$NAME.pid
  printf "done\n"
}

status() {
  status_of_proc -p /var/run/$NAME.pid "" $NAME && exit 0 || exit $?
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    stop
    start
    ;;
  status)
    status
    ;;
  *)
    echo "Usage: $NAME {start|stop|restart|status}" >&2
    exit 1
    ;;
esac

exit 0
```


### Source Services

Name|Channels
---|---
Telnet|ooc, broadcast
EQLog|ooc, guild, auction, general, shout
PEQEditorSQLLog|peqeditorsqllog

### Broadcast Services

Name|Channels
---|---
Discord|ooc, auction, general, peqeditorsqllog
Telnet|ooc


### Service Descriptions

* Telnet - EQEMU uses this as a way to communicate with the server
* EQLog - Everquest's client generates a log when you type /log, and it logs data the client sees
* PEQEditorSQLLog - EQEMU's PEQ Editor is configured to log sql events, you can relay this info to discord
* Discord - Chat service that lets you relay information to it via bots


### Example of using sql:

```toml
# SQL Report can be used to show stats on discord
# An ideal way to set this up is create a private voice channel
# Then bind it to various queries

[sql_report]
	# Enable SQL Reporting
	enabled = false

	# host for database
	# default: 127.0.0.1:3306
	host = "127.0.0.1:3306"

	# username to connect to database with.
	# default: eqemu
	username = "eqemu"

	# password to connect to database with.
	# default: eqemupass
	password = "eqemupass"

	# database to connect to
	# default: eqemu
	database = "eqemu"


[[sql_report.entries]]
	channel_id = "678525065905831968"
	query = "SELECT level2 FROM character_data cd INNER JOIN account a ON a.id = cd.account_id WHERE a.status = 0 ORDER BY level2 DESC LIMIT 1"
	pattern = "Highest Level: {{.Data}}"
	refresh = "5m"

[[sql_report.entries]]
	channel_id = "676283027361366026"
	query = "SELECT count(id) FROM character_data WHERE zone_id != 386 AND last_login > UNIX_TIMESTAMP()-3600"
	pattern = "In Dungeon: {{.Data}}"
	refresh = "60s"

[[sql_report.entries]]
	channel_id = "678525229672169472"
	query = "SELECT count(id) FROM character_data WHERE zone_id = 386 AND last_login > UNIX_TIMESTAMP()-3600"
	pattern = "In Hub: {{.Data}}"
	refresh = "60s"

[[sql_report.entries]]
	channel_id = "676282331627257856"
	query = "SELECT count(id) FROM account"
	pattern = "Accounts: {{.Data}}"
	refresh = "30m"
```

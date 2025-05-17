# ZEDhook

Now modified for Discord webhooks.

`zedhook` is a script that can act as the "mail" program for the ZFS Event Daemon or [ZED](http://louwrentius.com/the-zfs-event-daemon-on-linux.html).

Instead of sending an email when ZFS encounters an error, `zedhook` will POST JSON like this:

~~~
{
  "content": "ZFS checksum error for backup on zedhost ```ZFS has detected a checksum error:&#13;eid: 5788&#13;class: checksum&#13;host: zedhost&#13;time: 2016-06-12 20:48:03-0500&#13;vtype: disk&#13;vpath: /dev/sdd1&#13;vguid: 0x1A7CE1031E023015&#13;cksum: 2472&#13;read: 0&#13;write: 0&#13;pool: backup```"
}
~~~

Using the `wget` command (you must have `wget` installed):

~~~
wget -O- --header=Content-Type:application/json --post-data=...
~~~

To whatever endpoint you specify in ZED's `/etc/zfs/zed.d/zed.rc` like this:

~~~
##
# zed.rc
#
# This file should be owned by root and permissioned 0600.
##

##
# Email address of the zpool administrator for receipt of notifications;
#   multiple addresses can be specified if they are delimited by whitespace.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
# Disabled by default; uncomment to enable.
#
ZED_EMAIL_ADDR="zed@hook.com"

##
# Name or path of executable responsible for sending notifications via email;
#   the mail program must be capable of reading a message body from stdin.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
#
ZED_EMAIL_PROG="/usr/local/bin/zedhook"

##
# Command-line options for ZED_EMAIL_PROG.
# The string @ADDRESS@ will be replaced with the recipient email address(es).
# The string @SUBJECT@ will be replaced with the notification subject;
#   this should be protected with quotes to prevent word-splitting.
# Email will only be sent if ZED_EMAIL_ADDR is defined.
#
ZED_EMAIL_OPTS="-s '@SUBJECT@' https://discord.com/api/webhooks/<snip>/<snip>/"

##
# Minimum number of seconds between notifications for a similar event.
#
ZED_NOTIFY_INTERVAL_SECS=3600
~~~

You'll notice that I've set the `ZED_EMAIL_ADDR` to a dummy value so ZED will run `ZED_EMAIL_PROG`.

I've set the `ZED_EMAIL_PROG` to `/usr/local/bin/zedhook`, which means I've downloaded the `zedhook` script to `/usr/local/bin/`. You can put the `zedhook` script anywhere on your system, as long as you tell ZED about the location by setting `ZED_EMAIL_PROG` accordingly.

Finally, `ZED_EMAIL_OPTS` has had the address part set to a [Discord](https://discord.com/) webhook, but you can have yours POST to any endpoint ready to consume the JSON.

## Local logging

Additionally, `zedhook` will log all messages to your server local log directory at `/var/log/zedhook.log`.

If there are problems reported by ZED, the contents of that log file will contain entries that look like this:

~~~
|-------------------------- 2016-06-13 03:02:50 UTC --------------------------|
Subject: ZFS checksum error for backup on zedhost

ZFS has detected a checksum error:
eid: 5788
class: checksum
host: zedhost
time: 2016-06-12 20:48:03-0500
vtype: disk
vpath: /dev/sdd1
vguid: 0x1A7CE1031E023015
cksum: 2472
read: 0
write: 0
pool: backup
|-----------------------------------------------------------------------------|
~~~

This is the exact same information sent by POST as JSON to your endpoint, but all in one conveinent place, which is especially useful if the endpoint was down.

## Other uses

`zedhook` emulates the API of the linux [`mail`](http://linux.die.net/man/1/mail) program (to the limited extent that was needed for ZED).

It can accept arguments like:

~~~
zedhook -s "Some kind of message" https://discord.com/api/webhooks/<snip>/<snip>/ < somelogfile
~~~

So, perhaps it could be useful for replacing the mail program in other simple log programs.

## Authors

__Joel Kuzmarski__

* <http://twitter.com/leoj3n>
* <http://github.com/leoj3n>

__Matt Wall__

* <https://github.com/llawwehttam>

## License

[![MIT License](https://img.shields.io/:license-MIT-blue.svg?)](https://tldrlegal.com/l/mit)


    ▒███████▒▓█████ ▓█████▄  ██░ ██  ▒█████   ▒█████   ██ ▄█▀
    ▒ ▒ ▒ ▄▀░▓█   ▀ ▒██▀ ██▌▓██░ ██▒▒██▒  ██▒▒██▒  ██▒ ██▄█▒ 
    ░ ▒ ▄▀▒░ ▒███   ░██   █▌▒██▀▀██░▒██░  ██▒▒██░  ██▒▓███▄░ 
      ▄▀▒   ░▒▓█  ▄ ░▓█▄   ▌░▓█ ░██ ▒██   ██░▒██   ██░▓██ █▄ 
    ▒███████▒░▒████▒░▒████▓ ░▓█▒░██▓░ ████▓▒░░ ████▓▒░▒██▒ █▄
    ░▒▒ ▓░▒░▒░░ ▒░ ░ ▒▒▓  ▒  ▒ ░░▒░▒░ ▒░▒░▒░ ░ ▒░▒░▒░ ▒ ▒▒ ▓▒
    ░░▒ ▒ ░ ▒ ░ ░  ░ ░ ▒  ▒  ▒ ░▒░ ░  ░ ▒ ▒░   ░ ▒ ▒░ ░ ░▒ ▒░
    ░ ░ ░ ░ ░   ░    ░ ░  ░  ░  ░░ ░░ ░ ░ ▒  ░ ░ ░ ▒  ░ ░░ ░ 
      ░ ░       ░  ░   ░     ░  ░  ░    ░ ░      ░ ░  ░  ░   
    ░                ░                                       


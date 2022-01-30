# Systemd Timers


## Basic problem

Cron is the 'good old standard' for periodic task management in Unix.

However, it is FAR from standard, can behave differently (or be configured differently) depending on service/config file/user.

How do you make a cron job? That depends on: cron, anacron, fcron, jobber, etc. My point is: this is not a steady, unified thing the way some Posix things or Unix traditions are, and it can be annoying depending on how complex your setup grows to be (logging, etc.).

So now I GENERALLY find myself using systemd timers, with good results.


### Benefits:

They're systemd units, so they can do everything other systemd units can:
- have dependencies (other units)
- fine granularity for triggering (hw state changes, other events systemd lets you react to)
- easy management via systemctl enable/disable/start/stop
- Timer events can be scheduled based on finish times of executions some delays can be set between executions.
- OnFailure
- EnvironmentFile and other systemd unit goodies
- systemcall filters
- user/group ids
- membershipcontrols
- nice value
- OOM score
- IO scheduling class and priority
- CPU scheduling policy CPU
- affinity umask
- timer slacks
- secure bits
- network access
- etc. etc. etc.


### Downsides

- can get a bit messy, lots of custom files instead of text in a single file.


## Basic structure

Cron job line consists of:
1. scheduling information
2. action information.

Systemd timers work the same way, but split each concept into its own file.
2 unit files: a service and a timer. One benefit you'll notice right away is that this lets us start/test these service units independently of their timers

### Create demo service

`vim /usr/local/bin/tutorialinux-systemd-timers.sh`

```
echo "I just ran on $(date)"
exit 0
```

### Create systemd service unit file

`vim /etc/systemd/system/tutorialinux.service`


```
[Unit]
Description=Run the tutorialinux echo service

[Service]
ExecStart=/usr/local/bin/tutorialinux-systemd-timers.sh
```


### Create systemd timer unit file

`vim /etc/systemd/system/tutorialinux.timer`

```
[Unit]
Description=My job timer

[Timer]
OnBootSec=0min
OnCalendar=*:*:0/30
Unit=my_job.service

[Install]
WantedBy=multi-user.target
```

## Cool features: testing

Test your time formats without saving/re-running your cronjob a bunch of times.

systemd-analyze calendar *:*:0/30

Based on: `man systemd.time`


### Turn it on!

Enable the timer and you're good to go!
`systemctl enable tutorialinux.timer`



## Logging

Just like any other systemd unit -- check the journal!
`journalctl -u tutorialinux.service`


## Sources

- [Official Docs](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
- [trstringer blog](https://trstringer.com/systemd-timer-vs-cronjob/)
- [Timers vs Cronjobs - stackexchange](https://unix.stackexchange.com/questions/278564/cron-vs-systemd-timers)


# SSH Hardening

## /path/to/ssh_monitor.sh

```shell
#!/bin/bash

while true; do
  users=$(who | awk '/pts/{print $1}' | sort -u)
  
  if [[ ! -z "$users" ]]; then
    for user in $users; do
      curl -s -o /dev/null "<uptime-kuma-push-url>?status=down&msg=$user"
    done
  else
    curl -s -o /dev/null "<uptime-kuma-push-url>?status=up"
  fi
  sleep 45
done
```

## Service file

```
[Unit]
Description=Monitor SSH logins and call curl
After=network.target

[Service]
ExecStart=/path/to/ssh_monitor.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

## Login shell script

```shell
#! /bin/bash

USER=$(whoami)
curl -s -o /dev/null "<uptime-kuma-push-url>?status=down&msg=${USER}"
if [[ -n $SSH_ORIGINAL_COMMAND ]]
then
    exec /bin/bash -c "$SSH_ORIGINAL_COMMAND"
else 
    exec bash -il
fi
```

[ssh-audit python module](https://pypi.org/project/ssh-audit/)
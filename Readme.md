

### Extend the terminal idle time / disable automatic logout

- change the sshd config `sudo nano /etc/ssh/sshd_config`
- add or modify the following 
- Time (in seconds) between server checks to see if the client is still connected. `ClientAliveInterval 0` 
- Number of server checks before considering the session dead. Setting this to 0 means the server will not check. `ClientAliveCountMax 0`
- restart the ssh service `systemctl restart ssh` or `systemctl restart sshd`


### Automated backup with cron 

- set up a cron task `crontab -e`
- example of task at 4am every morning 

```bash
# m h  dom mon dow   command
# everyday at 4 am and save output in a log file
0 4 * * * /var/www/totwebsite/wp-content/themes/maivdigital/db/db_dump.sh >> /var/www/totwebsite/wp-content/themes/maivdigital/db/dailylogfile.log 2>&1 
```


#### Truncate the logfile 

```bash
#!/bin/bash

LOG_FILE="dailylog.log"
MAX_SIZE=2  # Max file size in MB

# Check if the file exists
if [ -f "$LOG_FILE" ]; then
    # Get the file size in MB
    FILE_SIZE_MB=$(du -m "$LOG_FILE" | cut -f1)

    echo "File '$LOG_FILE' exists. Size: ${FILE_SIZE_MB}MB"

    # Check if the file size is greater than the max size
    if [ "$FILE_SIZE_MB" -gt "$MAX_SIZE" ]; then
        echo "File size exceeds ${MAX_SIZE}MB. Truncating..."
        > "$LOG_FILE"
        echo "File truncated."
    else
        echo "File size is within the limit."
    fi
else
    echo "File '$LOG_FILE' not found."
fi
```

#### List files in MB 

- `ls -lh` 
- `ls -lh | awk '{print $5, $9}'`
- `du -ah --max-depth=1 | awk '{printf "%.2f MB\t%s\n", $1/1024, $2}'`


#### Change the default Ubuntu file type color display
- check if the default color is set `echo $LS_COLORS`
- check if any color is assigned for a specific filetype. example .log file type
```bash
echo $LS_COLORS | grep -oE '\*\.log=[^:]+' || echo "No color set for .log files"

```
- create custom .dircolors file
- create a copy of the default color settings `dircolors -p > ~/.dircolors`
- edit  .dircolors with you own colors
- example `.log 04;33` - will create bold underlined yellow color for .log file type
- more color codes:
        - 31 → Red

        - 32 → Green

        - 34 → Blue

        - 35 → Magenta

        - 36 → Cyan

        - 90 → Gray (default for backup files)

- apply new color `eval "$(dircolors -b ~/.dircolors)"`
- `source ~/.bashrc`


### Ubuntu server Alias settings

```bash
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias cls='clear'
alias showmb='ls -lh | awk "{print $5, $9}"'

# Apache
alias atest='apache2ctl configtest'
alias astatus='systemctl status apache2'
alias areload='systemctl reload apache2'
alias arestart='systemctl restart apache2'

# Git 
alias gs='git status'
alias pull='git pull'
alias push='git push origin master'
alias commit='git commit -m "update"'

```
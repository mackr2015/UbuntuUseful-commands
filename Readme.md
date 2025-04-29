

### Extend the terminal idle time / disable automatic logout

- change the sshd config `sudo nano /etc/ssh/sshd_config`
- add or modify the following 
- Time (in seconds) between server checks to see if the client is still connected. `ClientAliveInterval 0` 
- Number of server checks before considering the session dead. Setting this to 0 means the server will not check. `ClientAliveCountMax 0`
- restart the ssh service `systemctl restart ssh` or `systemctl restart sshd`


### TAR with GZ 

- make a tar with gz `tar -czvf archive.tar.gz -C /path/to/directory .`
- extract the tar.gz to specific location `tar -xzvf archive.tar.gz -C /your/target/directory/`


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


### Increase the File size upload limit
- run this to locate the loaded php configuration file
- `php --ini | grep "Loaded Configuration File"`
- this should display this file `/etc/php/your-version/php.ini` , also check this path `/etc/php/your-version/apache2/php.ini`
- look for variable `upload_max_filesize`
- restart apache `systemctl restart apache2`


### Fresh Ubuntu install is missing Imagick or GD PHP extensions ### 

- this needs to be installed in Ubuntu
- `apt install php-imagick`  and `apt install php-gd`
- installation for a specific PHP version `apt install php8.0-imagick php8.0-gd`
- systemctl restart apache2

### DB Search & Replace WP CLI

- use the WP CLI commands to perform search & replace 
- This command performs search and replace with addition flags and stores the results in the separate file
- `wp search-replace 'old_string' 'new_string' --precise --all-tables --export=path/to/your/file.sql`


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
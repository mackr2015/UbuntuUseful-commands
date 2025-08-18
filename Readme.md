
### GitHub SSH key available on the server so you can git pull (or push) securely via SSH — and keep it working after reboots.

- Copy your private SSH key for github to your server 
`scp ~/.ssh/id_rsa root@164.125.1251:~/.ssh/github_id_rsa`
- On the server set strict permissions `cmod 600 ~/.ssh/github_id_rsa`
- Create or update your SSh config on the server
`sudo nano ~/.ssh/config` 
- add this: 
```bash
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_id_rsa
    IdentitiesOnly yes
```
- Make sure the SSH agent loads your key on login. Add this to server's `~/.bashrc` or `~/.bash_profile`

```bash
# Start ssh-agent if not running, and add GitHub key
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/github_id_rsa
fi
```
- test this setup when logged in to your server run `ssh -T git@github.com`
- you should see welcome message from GitHub. 


# Enable a new virtual host site and install Certbot

- make sure that A record for domain and www is pointing to the correct IP
- when the virtual host file is ready `yoursite.conf` enable it
- `a2ensite yoursite.conf` 
- check the apache syntax `apache2ctl configtest` and reload apache
- `sudo certbot --apache -d yoursite.com -d www.yoursite.com`
- if certbot fails run `sudo certbot --apache --debug-challenges`
- or list all loaded vhosts `apache2ctl -S`

# MySQL settings a new user and new database

- create a new user `mysql -uroot -p`
- list all users `SELECT User, Host FROM mysql.user;`
- 
```sql
-- Create the user
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'user_password';

-- Grant all privileges on a specific database
GRANT ALL PRIVILEGES ON database_name.* TO 'newuser'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

```
- import db `mysql -u username -p database_name < /path/to/your/file.sql`


### DB Search & Replace WP CLI

- use the WP CLI commands to perform search & replace 
- This command performs search and replace with addition flags and stores the results in the separate file
- Instead of applying changes to the live database, it exports the modified SQL to a new path specified below. Live database remains unchanged.
- `wp search-replace 'old_string' 'new_string' --precise --all-tables --export=path/to/your/file.sql`


### Check the disk size for a specific directory
- explanation: 
    -h: Human readable (MB/GB)

    --max-depth=1: Only show top-level directories

    sort -hr: Sort by size, largest first

    head -n 10: Show top 10 entries
- ` sudo du -h /your-directory-path --max-depth=1 | sort -hr | head -n 10`


#### List files in MB 

- `ls -lh` 
- `ls -lh | awk '{print $5, $9}'`
- `du -ah --max-depth=1 | awk '{printf "%.2f MB\t%s\n", $1/1024, $2}'`



### Extend the terminal idle time / disable automatic logout

- change the sshd config `sudo nano /etc/ssh/sshd_config`
- add or modify the following 
- Time (in seconds) between server checks to see if the client is still connected. `ClientAliveInterval 0` 
- Number of server checks before considering the session dead. Setting this to 0 means the server will not check. `ClientAliveCountMax 0`
- restart the ssh service `systemctl restart ssh` or `systemctl restart sshd`


### TAR with GZ 

- c → create a new archive

- z → compress with gzip

- v → verbose output (optional, shows progress)

- f archive-name.tar.gz → name of the output file

- make a tar with gz `tar -czvf archive.tar.gz -C /path/to/directory .`
- make a tar file for multiple directories `tar -czvf arachive.tar.gz dir1 dir2 dir3`
- extract the tar.gz to specific location `tar -xzvf archive.tar.gz -C /your/target/directory/`
- tar with gz and specify multiple directories that you want to skip `tar -czf archive.tar.gz --exclude='folder1' --exclude='folder2' --exclude='path/to/folder3' my_project/` path is relative from where you do the tar command, so no need for full path like `/var/www/websitename`
- to extract tar.gz file `tar -xzvf your_file.tar.gz`
- to extract to specific location `tar -xzvf your_file.tar.gz -C /path/to/destination/`


### Automated backup with cron 

- set up a cron task `crontab -e`
- example of task at 4am every morning 

```bash
# m h  dom mon dow   command
# everyday at 4 am and save output in a log file
0 4 * * * /var/www/totwebsite/wp-content/themes/maivdigital/db/db_dump_cron.sh >> /var/www/totwebsite/wp-content/themes/maivdigital/db/dailylogfile.log 2>&1 
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

### Apache server Conf

- enable a new apache conf file `a2ensite myproject.conf` - this will create a symlink in `/etc/apache2/sites-enabled/`





### Ubuntu server Alias settings

```bash
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias cls='clear'
alias showmb="du -ah --max-depth=1 | awk '{printf "%.2f MB\t%s\n", $1/1024, $2}'"

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
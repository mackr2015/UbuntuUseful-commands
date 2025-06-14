## WP site Deployment steps


### Backup/Copy files and code

- Domain DNS point
    - point the domain DNS A to a server IP
- Apache/ Nginx
    - backup `/etc/apache2/sites-available/your-sitename.conf` to `/etc/apache2/sites-available/your-sitename.conf.back`
- Adjust or Install a new SSL
    - install a new SSL certificate for a new domain `/etc/apache2/sites-available/your-sitename.conf`
- DB
    - export the database and save a copy. Find a db dump stored and run the bash script for a fresh copy 
    - from the wordpress root path run wp cli command for search-replace `wp search-replace 'old_string' 'new_string' --precise --all-tables --export=path/to/your/file.sql`
    - manually check and correct if there are any references left with the old string
- Tar GZ the WP site and theme
    - tar the wp site, uploads, theme
    - make a tar with gz `tar -czvf archive.tar.gz -C /path/to/directory .`
    - extract the tar.gz to specific location `tar -xzvf archive.tar.gz -C /your/target/directory/`

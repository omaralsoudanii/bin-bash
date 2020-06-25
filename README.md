![bin-bash logo](https://user-images.githubusercontent.com/7079173/85646214-4a8f0480-b6a4-11ea-978e-290ed4017129.png)

This repo contains an incremental list of useful linux shell commands/scripts for future reference, covering the following tools:

## Table of Contents
- [General](#General)
- [Docker](#Docker)
- [MySQL](#MySQL)
- [MongoDB](#MongoDB)
- [PHP](#PHP)

## General

- **<h4>Find out how many lines a file contains a phrase:</h4>**
    ```bash
    $ cat /var/log/app.log.2 | grep "api/users" | wc -l
    ```

- **<h4>Check all mounted hard drives disk usage size:</h4>**
    ```bash
    $ df -h
    ```

- **<h4>Check current directory disk usage size:</h4>**
    ```bash
    $ du -sh *
    ```

- **<h4>Tail all incoming new lines for log file(s):</h4>**
    ```bash
    $ tail -f -n0 *-error_logs.log
    ```

- **<h4>Find if file(s) exists in the current directory/sub-directories, and list found file(s) locations:</h4>**
    ```bash
    $ find . -name "*-logs.json" | xargs ls -lh
    ```
  
    **Note:** This needs sudo to look in directories that requires root permission, it will also list of all files and directories in the current directory if it finds nothing.

- **<h4>Find if process(es) are currently running, and list process(es) details (parent/child) if found:</h4>**
    ```bash
    $ ps aux | grep mysql
    ```

- **<h4>Start/Restart a service with a different user:</h4>**
    ```bash
    $ sudo -u www-data service nginx start
    $ sudo -u omar service netdata restart
    ```

- **<h4>Add a script that automatically activates SSH agent on the shell startup for new user sessions:</h4>**
    -    **<h5>Edit your .bashrc OR .profile file (depending on your distro, but it is probably .bashrc):</h5>**
            ```bash
            $ sudo nano ~/.bashrc
            ```
    -    **<h5>Add the following lines at the bottom of the file:</h5>**
            ```bashell
            if [ ! -S ~/.ssh/ssh_auth_sock ]; then
              eval `ssh-agent`
              ln -sf "$SSH_AUTH_SOCK" ~/.ssh/ssh_auth_sock
            fi
            export SSH_AUTH_SOCK=~/.ssh/ssh_auth_sock
            ssh-add -l > /dev/null || ssh-add
            ```
    -    **<h5>Reload your .bashrc OR .profile file config:</h5>**
            ```bash
            $ source ~/.bashrc
            $ bash
            ```

            **Note:** The above steps will run a script that will activate SSH agent every time you acquire a new user session (restart/logout) and opening the terminal for the first time in the current session, which requires root password, so you will see a password prompt to activate. 
            After entering your root password you can SSH into any server you have access to, without the need to enter your credentials every time.

## Docker

- **<h4>Build a docker compose file images and run their containers assuming default compose file name (docker-compose.yml):</h4>**
    ```bash
    $ docker-compose up  --build -d
    ```

- **<h4>Build a docker compose file images and run their containers found in the specified path:</h4>**
    ```bash
    $ docker-compose  -f docker-compose.development.yml up  --build -d
    ```

- **<h4>Build a docker compose file images and run their containers while ignoring pre images cache (useful for multi-stage builds):</h4>**
    ```bash
    $ docker-compose build  --no-cache
    ```

- **<h4>Execute a command from a container:</h4>**
    ```bash
    $ docker exec -it node_app npm install
    ```

- **<h4>Export MySQL database from a container to a directory using mysqldump and execute command:</h4>**
    -    **<h5>Full database structure and data:</h5>**
            ```bash
            $ docker exec -i app_mysql-server-57_1 mysqldump -uroot -proot --databases mydb_name --skip-comments > /Documents/db.sql
            ```
    -    **<h5>Specified table structure and data:</h5>** 
            ```bash
            $ docker exec -i app_mysql-server-57_1 mysqldump -uroot -proot mydb_name mytable_name > /Documents/db.sql
            ```
    -    **<h5>Specific table structure and data that matches a where clause:</h5>** 
            ```bash
            $ docker exec app_mysql mysqldump -uroot -proot mydb_name --tables mytable_name --where="id>5 limit 5000000" > mytable_name.sql
            ```

- **<h4>SSH into a container using execute command:</h4>**
    -    **<h5>For alpine images, only **sh** is available:</h5>** 
            ```bash
            $ docker exec -it app_server sh
            ```
    -    **<h5>For other images, **bash** is available:</h5>**
            ```bash
            $ docker exec -it app_server bash
            ```

- **<h4>Preview a docker compose file configuration found in the specified path:</h4>**
    ```bash
    $ docker-compose -f docker-compose.development.yml config
    ```

- **<h4>Delete all stopped containers and unused images/networks/volumes and non persistent volumes and their networks:</h4>**
    ```bash
    $ docker system prune -a
    ```
    **Note:** Beware that all stopped containers, and unused images/networks/volumes will be deleted.

- **<h4>Delete all unused networks:</h4>**
    ```bash
    $ docker network prune
    ```

- **<h4>Delete all unused volumes:</h4>**
    ```bash
    $ docker volume ls -qf dangling=true | xargs -r docker volume rm
    ```

- **<h4>Tail container logs:</h4>**
    ```bash
    $ docker logs --tail 100 -f app_server
    ```

- **<h4>Tail container logs, and find lines that contains a phrase and print the containing line, 3 lines before it and 3 lines after it:</h4>**
    ```bash
    $ docker logs -f --tail 10000  app_server | grep -B3 -A3 "api/users"
    ```

- **<h4>System wide information for Docker:</h4>**
    ```bash
    $ docker system info
    ```

- **<h4>Docker disk usage info for the whole machine (detailed and non detailed):</h4>**
    ```bash
    $ docker system df -v
    ```
    ```bash
    $ docker system df
    ```

## MySQL

- **<h4>Export MySQL database to a directory using mysqldump:</h4>**
    -    **<h5>Full database structure and data:</h5>** 
            ```bash
            $ mysqldump -uroot -proot --databases mydb_name --skip-comments > /Documents/db.sql
            ```
    -    **<h5>Specified table structure and data:</h5>** 
            ```bash
            $ mysqldump -uroot -proot mydb_name mytable_name > /Documents/db.sql
            ```
    -    **<h5>A specific table structure and data that matches a where clause:</h5>** 
            ```bash
            $ mysqldump -uroot -proot mydb_name --tables mytable_name --where="id>5 limit 5000000" > mytable_name.sql
            ```

- **<h4>Change MySQL CLI pager to **less** for more readability when viewing large sets of data from queries in the terminal (use this when you are inside mysql shell i.e: after using **mysql** command):</h4>**
    ```bash
    $ pager less -R -S
    ```

- **<h4>Enable and use profiling in the current mysql session for debugging and tuning queries performance:</h4>**
    ```sql
    SET profiling = 1;
    select * from table_name where field_name like '%omar%';
    SHOW PROFILES;
    SHOW PROFILE FOR QUERY 1; ## 1 is the ID from SHOW PROFILES
    SHOW PROFILE CPU FOR QUERY 2;
    ```

## MongoDB

- **<h4>Show all databases:</h4>**
    ```mongojs
    show dbs
    ```

- **<h4>Select a databases to work on:</h4>**
    ```mongojs
    use mydb_name
    ```

- **<h4>Show all collections for a selected database:</h4>**
    ```mongojs
    show collections
    ```

- **<h4>Create a new collection in a selected database:</h4>**
    ```mongojs
    db.createCollection("collection_name")
    ```

- **<h4>Get all indexes for a collection:</h4>**
    ```mongojs
    db.collection_name.getIndexes()
    ```

- **<h4>Create a new index on a field in a collection:</h4>**
    ```mongojs
    db.collection_name.createIndex({field_name:1})
    ```

- **<h4>Drop an index from a field in a collection:</h4>**
    ```mongojs
    db.collection_name.dropIndex({field_name:1})
    ```

## PHP

- **<h4>Run a script using specific PHP version, and passing environment variables to the script:</h4>**
    ```bash
    $ WEBSITE=www.reddit.com ENVIROMENT=development  /usr/bin/php7.4 -f /var/www/swoole/server.php 
    ```
    
- **<h4>Enable/Disable PHP modules for a specific PHP version (needs root permission for basic modules):</h4>**
    -    **<h5>Enable:</h5>**
            ```bash
            $ sudo phpenmod -v 7.4 xdebug
            ```
    -    **<h5>Disable:</h5>** 
            ```bash
            $ sudo phpdismod -v 7.4 curl
            ```

- **<h4>Check the PHP settings of your installed version (In case you are using multiple PHP versions on the same machine, use ```which php``` and ```php -v``` to find the used version and its binary location):</h4>**
    ```bash
    $ php -i
    $ php -i | grep "error_reporting"
    ```

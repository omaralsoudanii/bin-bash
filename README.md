# An incremental list of useful linux shell commands/scripts for future reference, covering the following tools:

- [General](#General)
- [Docker](#Docker)
- [MySQL](#MySQL)
- [MongoDB](#MongoDB)
- [PHP](#PHP)

## General

- Find out how many lines a file contains a phrase:
    ```sh
    cat /var/log/app.log.2 | grep "api/users" | wc -l
    ```

- Check all mounted hard drives disk usage size:
    ```sh
    df -h
    ```

- Check current directory disk usage size:
    ```sh
    du -sh *
    ```
- Tail all incoming new lines for log file(s):
    ```sh
    tail -f -n0 *-error_logs.log
    ```
    
- Find if file(s) exists in the current directory/sub-directories, and list found file(s) locations:
    ```sh
    find . -name "*-logs.json" | xargs ls -lh
    ```
    **This needs sudo to look in directories that requires root permission, it will also list of all files and directories in the current directory if it finds nothing.**

- Find if process(es) are currently running, and list process(es) details (parent/child) if found:
    ```sh
    ps aux | grep mysql
    ```

- Start/Restart a service with a different user:
    ```sh
    sudo -u www-data service nginx start
    sudo -u omar service netdata restart
    ```

- Add a script that automatically activates SSH agent on the shell startup for new user sessions:
    1.    Edit your **.bashrc** OR **.profile** file (depending on your distro, but it is probably .bashrc):
            ```sh
            sudo nano ~/.bashrc
            ```
    2.    Add the following lines at the bottom of the file:
            ```shell
            if [ ! -S ~/.ssh/ssh_auth_sock ]; then
              eval `ssh-agent`
              ln -sf "$SSH_AUTH_SOCK" ~/.ssh/ssh_auth_sock
            fi
            export SSH_AUTH_SOCK=~/.ssh/ssh_auth_sock
            ssh-add -l > /dev/null || ssh-add
            ```
    3.    Reload your **.bashrc** OR **.profile** file config:
            ```sh
            source ~/.bashrc
            bash
            ```
            
  The above steps will run a script that will activate SSH agent every time you acquire a new user session (restart/logout) and opening the terminal for the first time in the current session, which requires root password, so you will see a password prompt to activate. 
  After entering your root password you can SSH into any server you have access to, without the need to enter your credentials every time.

## Docker

- Build a docker compose file images and run their containers assuming default compose file name (docker-compose.yml):
    ```sh
    docker-compose up  --build -d
    ```

- Build a docker compose file images and run their containers found in the specified path:
    ```sh
    docker-compose  -f docker-compose.development.yml up  --build -d
    ```

- Build a docker compose file images and run their containers while ignoring pre images cache (useful for multi-stage builds):
    ```sh
    docker-compose build  --no-cache
    ```

- Execute a command from a container:
    ```sh
    docker exec -it node_app npm install
    ```

- Export MySQL database from a container to a directory using mysqldump and execute command:
    1.    Full database structure and data: 
            ```sh
            docker exec -i app_mysql-server-57_1 mysqldump -uroot -proot --databases mydb_name --skip-comments > /Documents/db.sql
            ```
    2.    Specified table structure and data: 
            ```sh
            docker exec -i app_mysql-server-57_1 mysqldump -uroot -proot mydb_name mytable_name > /Documents/db.sql
            ```
    3.    Specific table structure and data that matches a where clause: 
            ```sh
            docker exec app_mysql mysqldump -uroot -proot mydb_name --tables mytable_name --where="id>5 limit 5000000" > mytable_name.sql
            ```
    
- SSH into a container using execute command:
    1.    For alpine images, only **sh** is available: 
            ```sh
            docker exec -it app_server sh
            ```
    2.    For other images, **bash** is available: 
            ```sh
            docker exec -it app_server bash
            ```

- Preview a docker compose file configuration found in the specified path:
    ```sh
    docker-compose -f docker-compose.development.yml config
    ```

- Delete all stopped containers and images and non persistent volumes and their networks:
    ```sh
    docker system prune -a
    ```
    **BEWARE THAT ALL STOPPED CONTAINERS DATA WILL BE DELETED**

- Delete all unused networks:
    ```sh
    docker network prune
    ```

- Delete all unused volumes:
    ```sh
    docker volume ls -qf dangling=true | xargs -r docker volume rm
    ```

- Tail container logs:
    ```sh
    docker logs --tail 100 -f app_server
    ```

- Tail container logs, and find lines that contains a phrase and print the containing line, 3 lines before it and 3 lines after it:
    ```sh
    docker logs -f --tail 10000  app_server | grep -B3 -A3 "api/users"
    ```

- System wide information for Docker:
    ```sh
    docker system info
    ```

- Docker disk usage info for the whole machine (detailed and non detailed):
    ```sh
    docker system df -v
    ```
    ```sh
    docker system df
    ```

## MySQL

- Export MySQL database to a directory using mysqldump:
    1.    Full database structure and data: 
            ```sh
            mysqldump -uroot -proot --databases mydb_name --skip-comments > /Documents/db.sql
            ```
    2.    Specified table structure and data: 
            ```sh
            mysqldump -uroot -proot mydb_name mytable_name > /Documents/db.sql
            ```
    3.    A specific table structure and data that matches a where clause: 
            ```sh
            mysqldump -uroot -proot mydb_name --tables mytable_name --where="id>5 limit 5000000" > mytable_name.sql
            ```

- Change MySQL CLI pager to **less** for more readability when viewing large sets of data from queries in the terminal (use this when you are inside mysql shell i.e: after using **mysql** command):
    ```sh
    pager less -R -S
    ```

- Enable and use profiling in the current mysql session for debugging and tuning queries performance:
    ```sql
    SET profiling = 1;
    select * from table_name where field_name like '%omar%';
    SHOW PROFILES;
    SHOW PROFILE FOR QUERY 1; ## 1 is the ID from SHOW PROFILES
    SHOW PROFILE CPU FOR QUERY 2;
    ```

## MongoDB

- Show all databases:
    ```json
    show dbs
    ```

- Select a databases to work on:
    ```json
    use mydb_name
    ```

- Show all collections for a selected database:
    ```json
    show collections
    ```

- Create a new collection in a selected database:
    ```json
    db.createCollection("collection_name")
    ```

- Get all indexes for a collection:
    ```json
    db.collection_name.getIndexes()
    ```

- Create a new index on a field in a collection:
    ```json
    db.collection_name.createIndex({field_name:1})
    ```

- Drop an index from a field in a collection:
    ```json
    db.collection_name.dropIndex({field_name:1})
    ```

## PHP

- Run a script using specific PHP version, and passing environment variables to the script:
    ```sh
    WEBSITE=www.reddit.com ENVIROMENT=development  /usr/bin/php7.4 -f /var/www/swoole/server.php 
    ```
    
- Enable/Disable PHP modules for a specific PHP version (needs root permission for basic modules):
    1.    Enable: 
            ```sh
            sudo phpenmod -v 7.4 xdebug
            ```
    2.    Disable: 
            ```sh
            sudo phpdismod -v 7.4 curl
            ```
- Check the PHP settings of your installed version (In case you are using multiple PHP versions on the same machine, use ```which php``` and ```php -v``` to find the used version and its binary location):
    ```sh
    php -i
    php -i | grep "error_reporting"
    ```

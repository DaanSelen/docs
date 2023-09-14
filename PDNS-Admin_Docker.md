# PowerDNS-Admin Docker Container Guide:

Author: **Daan Selen**.<br>
For the source-code, refer to the **end of the page**.

This tutorial guides a reader through installing PowerDNS-Admin, through using a docker-compose (sometimes known as a Portainer stack).

## **Prerequisites:**
* A healthy running instance of PowerDNS, with access to its REST API and corresponding API-Key.
* A healthy running database server (SQL-compatible e.g. MySQL-Server, MariaDB-Server or PostgreSQL).
* An updated and ready-to-use docker environment with access to the 'docker' command.

## **Installation using Docker-Compose**
### 1. Preparing the docker-compose.

Configure the docker-compose.yml as desired.
```
version: "3"

services:
  app:
    image: powerdnsadmin/pda-legacy:latest     #Available tags include: 'dev' and specific versions e.g. '0.3'.
    container_name: powerdns_admin             #The name the container will be given after deployment.
    ports:                                     #The assigned external:internal ports, by default PDNS-Admin uses 80 (HTTP).
      - "9191:80"
    logging:
      driver: json-file
      options:
        max-size: 50m
    environment:                                                                                        #Environment variables such as for connecting to the PDNS-Admin database server and database.
      - SQLALCHEMY_DATABASE_URI=postgresql://<dbuser>:<dbpassword>@<ip-address>:<port>/<database-name>  #This example assumes a PostgreSQL database instance.
      - GUNICORN_TIMEOUT=60                                                                             #For the Gunicorn parameters refer to the document below.
      - GUNICORN_WORKERS=2
      - GUNICORN_LOGLEVEL=DEBUG
```
Gunicorn parameters: [Gunicorn Settings Documentation](https://docs.gunicorn.org/en/stable/settings.html)<br>
Information about DATABASE URI: [Metacpan Explanation](https://metacpan.org/pod/URI::db)

### 2. Running the docker-compose file (detached/daemonized).

Using the previously prepared docker-compose.yml file we can run it in the background using:

```
docker-compose -f docker-compose.yml up -d
```
Flags:
* -f specifies the docker-compose file to use. In this example: docker-compose.yml.
* -d detaches the proces to make it run in the background. Remove this flag to see the logs in the foreground.

To check if the docker container is running:
```
docker-compose ps
```
An output like the example below would confirm a succesful execution.
```
     Name                   Command                       State                  Ports        
----------------------------------------------------------------------------------------------
powerdns_admin   entrypoint.sh gunicorn pow ...   Up (health: starting)   0.0.0.0:9191->80/tcp
```

### 3. Check PowerDNS-Admin Web-GUI.

In the case that everything is configured correctly there will be a login page present, comparable to the image below:

![PDNS-Admin_LoginPage](https://cloud.nerthus.nl/remote.php/dav/files/nerthuspublic/md/PDNS-Admin_Docker/PDNS-Admin_LoginPage.png)

### 4. Configuring the PowerDNS-Admin to PowerDNS-Server connection.

Scrolling down in the Web-GUI and navigating through Settings > Server you will find the fields needed to connect PowerDNS-Admin to a PowerDNS-Server.

To  see your PowerDNS-API Version you can check that at: 
```
curl -H 'X-API-Key: <API-KEY>' <PowerDNS-Server Host>:<Port>/api/v1/servers/localhost
```

## References

[Source Code](https://github.com/PowerDNS-Admin/PowerDNS-Admin)
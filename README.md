# Docker-compose

Example of using docker-compose to deploy a load-balanced web application


## The file docker-compose.yaml

```yaml
$ cat docker-compose.yaml

version: '3'

x-common-variables: &common-variables
  MYSQL_DATABASE: westwing
  MYSQL_USER: MYSQL_USER
  MYSQL_PASSWORD: MYSQL_PASSWORD

services:
  percona_db:
    image: percona
    restart: always
    volumes:
      - ./percona/setup.sql:/etc/mysql/init/setup.sql
    ports:
      - "13306:3306"
    environment:
      <<: *common-variables
      MYSQL_ROOT_PASSWORD: MYSQL_ROOT_PASSWORD
      MYSQL_HOST: localhost

  webapp1:
    image: node  
    restart: always
    depends_on:
      - percona_db
    ports:
      - "10000:10000"
    volumes:
      - ./webapp:/var/webapp/
    environment:
      <<: *common-variables
      MYSQL_HOST_IP: percona_db      
      - PORT=10000
      - SERVER_ID=1

  webapp2:
    image: node  
    restart: always
    depends_on:
      - percona_db
    ports:
      - "10001:10001"
    volumes:
      - ./webapp:/var/webapp/
    environment:
      <<: *common-variables
      MYSQL_HOST_IP: percona_db      
      - PORT=10001
      - SERVER_ID=2
      
  nginx:
    image: nginx
    restart: always    
    depends_on:
      - webapp1
      - webapp2
    ports:
      - "18080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    command: ["nginx", "-c", "/etc/nginx/nginx.conf",  "-g", "daemon off;"]
```
`sudo apt update`

`sudo apt install mariadb-server`
- Répondre aux questions de sécurisation :
`sudo mysql_secure_installation`

`sudo mysql` : 

```
CREATE DATABASE basename;
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL ON basename.* TO 'username'@'%';
FLUSH PRIVILEGES;
QUIT;
```



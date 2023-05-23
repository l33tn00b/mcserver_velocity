# mcserver_velocity
Set Up Minecraft Server

# Choose Server Image
- we'll go for Ubuntu 22.04 LTS

# Basic Server Setup
- on server:
  - ```apt-get install mc```
  - ```ufw allow ssh```
  - ```ufw enable```
  - install java as per: https://docs.papermc.io/misc/java-install
  - create new user: ```adduser mcrunner```
  - ```su mcrunner```
  - get current velocity version from: https://papermc.io/downloads/velocity
    -  ```mkdir velocity```
    -  ```cd velocity```
    -  ```wget <link>```
- on client:
  - ```ssh-keygen -t rsa -b 4096```, setting key file location/name: ```id_rsa_mc1```
  - copy key to server: ```ssh-copy-id -i /home/<local username>/.ssh/id_rsa_mc1.pub root@<your server name or ip>```
  - edit local ssh config: ```nano /home/<local username>/.ssh/config```, add lines:
    ```
    Host <hostname as per A/AAAA record>
        IdentitiesOnly yes
        IdentityFile /home/<local username>/.ssh/id_rsa_mc1
    ```
- DNS records
  -  Set A and AAAA records 

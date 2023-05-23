# mcserver_velocity
Set Up Minecraft Server, going from  ssh password login enabled Ubuntu image to velocity/paper based server with geyser.

# Choose Server Image
- we'll go for Ubuntu 22.04 LTS

# Basic Server Setup
- on server:
  - convenience: ```apt-get install mc```
  - get firewall up and running:
    - ```ufw allow ssh```
    - ```ufw enable```
- on client:
  - ```ssh-keygen -t rsa -b 4096```, setting key file location/name: ```id_rsa_mc1```
  - copy key to server: ```ssh-copy-id -i /home/<local username>/.ssh/id_rsa_mc1.pub root@<your server name or ip>```
  - edit local ssh config: ```nano /home/<local username>/.ssh/config```, add lines:
    ```
    Host <hostname as per A/AAAA record>
        IdentitiesOnly yes
        IdentityFile /home/<local username>/.ssh/id_rsa_mc1
    ```
- DNS records (wherever you may be able to do so...)
  -  Set A and AAAA records 

- on server:
  - disable password based login: ```nano /etc/ssh/sshd_config```: look for ```PasswordAuthentication yes```
    - change to 
      ```
      PasswordAuthentication no
      ```
    - save file.
    - restart ssh server: ```service ssh restart``` 
  - install java as per: https://docs.papermc.io/misc/java-install
  - create new user: ```adduser mcrunner```
  - ```su mcrunner```
  - ```cd ~```
  - get current velocity version from: https://papermc.io/downloads/velocity
    -  ```mkdir velocity```
    -  ```cd velocity```
    -  ```wget <link>```
    -  create shell script for starting velocity along the lines of: https://docs.papermc.io/velocity/getting-started
      -   ```touch start.sh```
      -   ```chmod u+x start.sh```
      -   edit file, paste contents: ```nano start.sh```
          ```
          #!/bin/sh

          java -Xms1G -Xmx1G -XX:+UseG1GC -XX:G1HeapRegionSize=4M -XX:+UnlockExperimentalVMOptions -XX:+ParallelRefProcEnabled -XX:+AlwaysPreTouch -XX:MaxInlineLevel=15 -jar velocity*.jar
          ```
  - test: ```./start.sh```, if all good, ```end```            
  - add geyser/floodgate for crossplay: https://wiki.geysermc.org/geyser/setup/
    - ```cd plugins```
    - ```wget https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/velocity -O Geyser-Velocity.jar```
    - ```wget https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/velocity -O floodgate-velocity.jar```

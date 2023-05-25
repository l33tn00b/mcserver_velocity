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
  - ```apt-get install net-tools``` (for netstat, checking port assigments of server(s) and proxy)
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
# Minecraft, on server
## Velocity proxy, Basics
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
        #java -Xms1G -Xmx1G -XX:+UseG1GC -XX:G1HeapRegionSize=4M -XX:+UnlockExperimentalVMOptions -XX:+ParallelRefProcEnabled -XX:+AlwaysPreTouch -XX:MaxInlineLevel=15 -jar velocity*.jar
        java -XX:+UnlockExperimentalVMOptions -XX:+ParallelRefProcEnabled -XX:+AlwaysPreTouch -XX:MaxInlineLevel=15 -jar velocity*.jar
        ```
- test: ```./start.sh```, if all good, ```end```            
## Crossplay with Geyser and Floodgate
- add geyser/floodgate for crossplay: https://wiki.geysermc.org/geyser/setup/
  - ```cd plugins```
  - ```wget https://download.geysermc.org/v2/projects/geyser/versions/latest/builds/latest/downloads/velocity -O Geyser-Velocity.jar```
  - ```wget https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/velocity -O floodgate-velocity.jar```
  - ```cd .. ```
  - start velocity proxy, creating plugin config files: ```./start.sh``` , quit velocity: ```end```   

## Paper as server
- download and install Paper: https://docs.papermc.io/paper/getting-started
- create dir for each server, install there
  - ```cd ~```
  - ```mkdir server1```
  - ```cd server1```
  - ```wget wget https://api.papermc.io/v2/projects/paper/versions/1.19.4/builds/538/downloads/paper-1.19.4-538.jar``` (or whatever is current...)
  - ```touch start.sh```
  - ```chmod u+x start.sh```
  - ```nano start.sh```, adding (leaving out memory min/max arguments because we have a small server):
    ```
    #!/bin/sh
    java -jar paper-1.19.4-538.jar --nogui
    ```
  - ```./start.sh``` will complain about eula acceptance, so 
  - ```nano eula.txt``` change to ```eula=true```, save, quit
  - ```./start.sh```wil run the server, ```stop```, ```quit``` to exit
  - install floodgate for crossplay:
    -  ```cd plugins```
    -  ```wget https://download.geysermc.org/v2/projects/floodgate/versions/latest/builds/latest/downloads/spigot -O floodgate-spigot.jar```
  - ```cd ..```
  - ```./start.sh```, check for plugin loading correctly, ```quit```
  - ```nano server.properties```
    - ```online-mode=false``` (because we'll rely on velocity doing the checks)
    - ```server-ip=127.0.0.1```
    - ```server-port=30067```
- optional: open a root ssh connection to server, check if paper is running (can't do that from outside the server because firewall is up):
  -  ```netstat -nlp``` output should contain a line
      ```
      tcp6       0      0 :::25565                :::*                    LISTEN      11563/java
      ```
      indicating a minecraft server listening to connections on port 25565. 
   - running a port scan from the outside will (hopefully) only show port 22:  
    - ```nmap -sS -sU -p 0-65535 -T4 -A -v <hostname>``` (this will take quite a while...)  
- edit global config file to accept velocity connections:
  - ```nano /home/mcrunner/server1/config/paper_global.yml```
    ```
     velocity:
       enabled: true
       online-mode: false
       secret: '<contents of /home/mcrunner/velocity/forwarding.secret>'
    ```
- set up auto-start:  
  - either ```sudo su``` (if you've given sudo privileges to user mcrunner (a little bit insecure...)) or from another root shell:
  - ```touch /etc/systemd/system/minecraft@paper1.service```
  - ```nano /etc/systemd/system/minecraft@paper1.service```, adding
    ```
    [Unit]
    Description=Paper Minecraft server 1
    After=network.target
    Wants=network-online.target

    [Service]
    Type=simple
    User=mcrunner
    Group=mcrunner
    WorkingDirectory=/home/mcrunner/server1
    ExecStart=/home/mcrunner/server1/start.sh
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```  
  - ```systemctl daemon-reload```
  - ```systemctl start minecraft@paper1.service```
  - ```ps -ef ```should show something like:
    ```
    mcrunner   12032       1  0 17:41 ?        00:00:00 /bin/sh /home/mcrunner/server1/start.sh
    mcrunner   12033   12032 15 17:41 ?        00:04:41 java -jar paper-1.19.4-538.jar --nogui
    ```
    all good, running as non-privileged user.
  - auto-start: ```systemctl enable minecraft@paper1.service```
  - in server console:
    - ```whitelist on```
    - ```whitelist add <username>```

## Finish Velocity Setup
- ```nano /home/mcrunner/velocity/velocity.toml```, changing:
  - ```player-info-forwarding-mode = "modern"```
  - ```kick-existing-players = true```
  - in [servers] section, comment out everything, add:
    ```
    # we'll leave port 30066 for a lobby, just in case
    paper1 = "127.0.0.1:30067"
    # In what order we should try servers when a player logs in or is kicked from a server.
    try = [
        "paper1"
    ]
    ```
  -  in [advanced] section, change:
    - ```tcp-fast-open = true```
    
## You should now be able to re-start velocity and paper

## Allow incoming connections
We'll allow incoming connections to our server's public IP address(es) only for specific ports (the ones handled by velocity (native Java) and its geyser plugin (bedrock clients))
- ```ufw allow proto udp from any to <your server's public ipv4> port 19132``` (bedrock clients)
- ```ufw allow proto tcp from any to <your server's public ipv4> port 25577``` (java clients)
- ```ufw allow proto udp from any to <your server's public ipv6> port 19132``` (bedrock clients)
- ```ufw allow proto tcp from any to <your server's public ipv6> port 25577``` (java clients)
- ```ufw reload```

# Admin tasks
## whitelisting java players
- go to paper server console
  - ```whitelist add <player name>```, you'll have an entry (which needs to be modified) in the whitelist file.
- get proper player UUID, e.g. from https://mcuuid.net by entering player name.
- quit server console or use another terminal session:
  - ```nano /home/mcrunner/server1/whitelist.json``` 
  - find the entry of the player to be whitelisted.
  - modify the UUID to be the one you just looked up.
  - save, quit (Ctrl-X, confirm)
- go back to server console
  - ```whitelist reload```

## whitelisting bedrock (=non-java) players
According to https://wiki.geysermc.org/geyser/faq/ (How do I add players to the whitelist when using Floodgate?).  
Bedrock client players will show up with a pre-pended ```.``` in front of their player names (if you didn't specify anything else).  
- ```fwhitelist add .<bedrock user name>``` (please note the pre-pended ```.```)


# Get your friends connected:
## Java, using TCP connection:
- Connect to ```<hostname>:25577```  
  output of ```netstat -nlp``` on your server:
  ```
  tcp6       0      0 :::25577                :::*                    LISTEN      26438/java
  ```

## Bedrock (crappy clients), using UDP connection
- Connect to ```<hostname>:19132```  
  output of ```netstat -nlp``` on your server:
  ```
  udp6       0      0 :::19132                :::*                                26438/java
  ``` 

# Get rid of featured servers?
- https://github.com/Pugmatt/BedrockConnect has a list of domain names to be blackholed (DNS) (pi-hole to the rescue)  
- perhaps you might also want to block the ip addresses of these servers in your network's firewall (hello, UDM) 
- do also blackhole: pocket.realms.minecraft.net
-  




<img src="/sas VA/Screenshot.PNG" alt="SAS VA Report" style="height: 600px; width:950px;"/>

- [Lijster Project - Read Gas and Electricity Usage using SAS ESP on an edge device](#lijster-project---read-gas-and-electricity-usage-using-sas-esp-on-an-edge-device)
  - [Install Ubuntu on BasPi](#install-ubuntu-on-baspi)
    - [Use correct images: arm or x64 linux? https://en.wikipedia.org/wiki/AArch64](#use-correct-images-arm-or-x64-linux-httpsenwikipediaorgwikiaarch64)
    - [version of linux?](#version-of-linux)
    - [Steps](#steps)
    - [Add user sas with pw sas](#add-user-sas-with-pw-sas)
    - [change hostname](#change-hostname)
  - [Install ESP on the Edge](#install-esp-on-the-edge)
    - [Get software](#get-software)
    - [install locally](#install-locally)
    - [ESP Server CLI start server on the edge](#esp-server-cli-start-server-on-the-edge)
  - [Install on docker](#install-on-docker)
    - [install.sh](#installsh)
    - [Extra:](#extra)
    - [troubleshooting:](#troubleshooting)
  - [test on BASPi docker](#test-on-baspi-docker)
  - [Monitor the project](#monitor-the-project)
  - [Reading thermometer data on BasPi:](#reading-thermometer-data-on-baspi)
  - [Add slimme meter http://www.gejanssen.com/howto/Slimme-meter-uitlezen/](#add-slimme-meter-httpwwwgejanssencomhowtoslimme-meter-uitlezen)
    - [Slimme meter](#slimme-meter)
    - [P1 Telegram](#p1-telegram)
- [Data processing and reporting](#data-processing-and-reporting)
  - [ESP Connect](#esp-connect)
    - [Use ESP connect](#use-esp-connect)
- [FTP](#ftp)
    - [use ftp server https://ubuntu.com/server/docs/service-ftp  in first version](#use-ftp-server-httpsubuntucomserverdocsservice-ftp--in-first-version)
- [Issues](#issues)
    - [Issue with ttyS0 --\> need to run after restart of BASPi](#issue-with-ttys0----need-to-run-after-restart-of-baspi)
    - [regex Temp](#regex-temp)
    - [fix issue with with TLS mixed content:](#fix-issue-with-with-tls-mixed-content)
    - [change time on Pi](#change-time-on-pi)
    - [issue with sudoers file:](#issue-with-sudoers-file)
    - [summer time correction (fixed)](#summer-time-correction-fixed)
    - [stop project from web interface (skip)](#stop-project-from-web-interface-skip)
    - [opening model from web interface (skip)](#opening-model-from-web-interface-skip)
    - [max power  (added)](#max-power--added)
    - [usage instead of meter readings  (done)](#usage-instead-of-meter-readings--done)
    - [Add thermostaat program  (done in VA Autoload)](#add-thermostaat-program--done-in-va-autoload)
    - [symbolic link from sas web server to ESP connect](#symbolic-link-from-sas-web-server-to-esp-connect)
      - [Symbolic link](#symbolic-link)
      - [Edit config C:\\SAS\\Config\\Lev1\\Web\\WebServer\\conf\\sas.conf](#edit-config-csasconfiglev1webwebserverconfsasconf)
      - [Use new links in VA: http://snlsea.emea.sas.com/esp-connect/examples/lijster/currentpower.html](#use-new-links-in-va-httpsnlseaemeasascomesp-connectexampleslijstercurrentpowerhtml)
    - [Open from outside the network](#open-from-outside-the-network)
    - [Archiving of data](#archiving-of-data)
    - [Implement secure ftp](#implement-secure-ftp)
    - [Add auto recovery from power failure and add Weekly restart to prevent memory issues](#add-auto-recovery-from-power-failure-and-add-weekly-restart-to-prevent-memory-issues)
      - [Add a startup ESP model to esp-properties.yml](#add-a-startup-esp-model-to-esp-propertiesyml)
      - [create startup script: startup](#create-startup-script-startup)
      - [schedule restart](#schedule-restart)
      - [enable startup script at restart](#enable-startup-script-at-restart)
- [TODO](#todo)
  - [ssl esp server](#ssl-esp-server)
  - [add mqtt](#add-mqtt)
- [TIPS](#tips)
    - [1. Linux](#1-linux)
    - [2. docker](#2-docker)
- [At startup of raspberry PI (this is now automated)](#at-startup-of-raspberry-pi-this-is-now-automated)
- [TIPS](#tips-1)
  - [1. Linux](#1-linux-1)
  - [2. docker](#2-docker-1)
- [Documentation](#documentation)


#  Lijster Project - Read Gas and Electricity Usage using SAS ESP on an edge device

## Install Ubuntu on BasPi 
### Use correct images: arm or x64 linux? https://en.wikipedia.org/wiki/AArch64
```
uname -m
```
### version of linux?  
```
cat /etc/os-release  
hostnamectl  
```
### Steps
1. Format micro SD to fat32 with Powershell: format /FS:FAT32 D:
2. Get ubuntu server image: for ARM https://ubuntu.com/download/raspberry-pi
3. Put it on micro sd Use Win32 Disk imager 
4. Install ssh https://likegeeks.com/ssh-connection-refused/
5. Install docker https://pimylifeup.com/ubuntu-install-docker/
6. Retrieve required files from SAS https://go.documentation.sas.com/doc/en/espcdc/v_005/dplyedge0phy0lax/p1goagdh3x1uqdn1m1rn57lngpud.htm
6.1 license: https://my.sas.com/en/my-orders.html
SASViyaV4_0B17MY_lts_2022.09_multipleAssets_2022-12-27T083512.zip
The zip file includes: SASViyaV4_0B17MY_certs.zip and SASViyaV4_0B17MY_lts_2022.09_license_2022-12-27T083512.jwt
6.2 mirror manager: https://support.sas.com/en/documentation/install-center/viya/deployment-tools/4/mirror-manager.html


### Add user sas with pw sas
```
sudo userdel sas 
sudo sudo adduser sas  
sudo groupadd sas  
sudo usermod -a -G docker sas dialout  
id sas  
newgrp docker 
``` 
- add user sas to dialout group  
```
usermod -aG sudoers
```

### change hostname
I use baspi.localdomain
https://www.cyberciti.biz/faq/ubuntu-18-04-lts-change-hostname-permanently/

## Install ESP on the Edge
### Get software
- Video from SAS: Daniel Kuiper / Rik de Ruiter 
https://communities.sas.com/t5/SAS-Communities-Library/Real-time-computer-vision-on-an-edge-device-2-ESP-edge/ta-p/788416
- You need a second Linux server to do this, I use an Ubuntu image on my laptop.
1. download certificates, licence, and mirror manager for linux (in C:\files\SourceTree\lijster\install)
2. copy them to the workshop image esp.localdomain
3. untar the mirrormgr-linux.tgz 
   cd /home/sas/edge/  
   tar -xvzf mirrormgr-linux.tgz  
4. move edge_mirror   
   mv esp-edge-extension/viya4/edge_mirror.sh .
5. chmod +x mirrormgr
   chmod +x edge_mirror.sh
6. download using 
   ./edge_mirror.sh aarch64-ubuntu-linux-16
7. move files from workshop image to pi
    

```
ssh sas@192.168.1.90 
mkdir deploy
exit
cd /home/sas/edge/
scp -r espedge_repos/ sas@172.28.225.213:~/deploy
scp -r SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt sas@172.28.225.213:~/deploy
```

New! for secured ssh use:
```
ssh -i ~/baspi82.ppk sas@192.168.1.90 -p 31422
mkdir deploy
exit
cd /home/sas/edge/
scp -P 31422 -i ~/baspi82.ppk -r espedge_repos sas@192.168.1.90:~/deploy
scp -P 31422 -i ~/baspi82.ppk -r ~/edge/SASViyaV4_0B17MY_lts_2022.09_license_2022-12-27T083512.jwt sas@192.168.1.90:~/deploy
```

### install locally
On pi:
```
sudo apt install /home/sas/deploy/espedge_repos/aarch64-ubuntu-linux-16/basic/*
sudo apt install /home/sas/deploy/espedge_repos/aarch64-ubuntu-linux-16/analytics/*
sudo apt install /home/sas/deploy/espedge_repos/aarch64-ubuntu-linux-16/astore/* 
sudo apt install /home/sas/deploy/espedge_repos/aarch64-ubuntu-linux-16/textanalytics/* 
sudo apt install /home/sas/deploy/espedge_repos/aarch64-ubuntu-linux-16/gpu/* 
```
- move license file 
```
ssh sas@192.168.1.90 
sudo mv /home/sasdeploy/SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt /opt/sas/viya/home/SASEventStreamProcessingEngine/etc/license/
```

New! for secured ssh use:
```
ssh -i ~/baspi82.ppk sas@192.168.1.90 -p 31422  
sudo mv /home/sas/deploy/SASViyaV4_0B17MY_lts_2022.09_license_2022-12-27T083512.jwt /opt/sas/viya/home/SASEventStreamProcessingEngine/etc/license/
```

### ESP Server CLI start server on the edge
```
export DFESP_HOME=/opt/sas/viya/home/SASEventStreamProcessingEngine/  
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DFESP_HOME/ssl/lib:$DFESP_HOME/lib:/opt/sas/viya/home/SASFoundation/sasexe:/usr/lib  
export PATH=$PATH:$DFESP_HOME/bin  
dfesp_xml_server -http 32415 -pubsub 32416 -loglevel "esp=error,common.http=debug,esp.windows=trace"  
```
- or
```  
nohup dfesp_xml_server -http 32415 -pubsub 32416 >/tmp/esp-server.log &
tail -f /tmp/esp-server.log
```
- stop server   
```
dfesp_xml_client -url "http://baspi:32415/eventStreamProcessing/v1/server/state?value=stopped" -put
```

## Install on docker

- See also:
- ESP with Docker Workshop (internal video)
https://web.microsoftstream.com/video/fc10393e-1753-4e09-8770-be98ca8c219e at 1:37:30
and
https://docs.docker.com/get-started/overview/
https://go.documentation.sas.com/doc/en/espcdc/v_031/dplyedge0phy0lax/p1goagdh3x1uqdn1m1rn57lngpud.htm

```
 apt install net-tools  
```
 
1. Create local registry  
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2  
```

2. Create docker folder en copy the files
```
cd ~   
mkdir docker  
cd docker  
cp -r ~/deploy/espedge_repos/aarch64-ubuntu-linux-16/ .  
cp -r "/opt/sas/viya/home/SASEventStreamProcessingEngine/etc/license/SASViyaV4_0B17MY_lts_2022.09_license_2022-12-27T083512.jwt" .  
```
3. package using  tar  
```
tar -cvzf dockeresp2022_9.tar ./aarch64-ubuntu-linux-16/ ./SASViyaV4_0B17MY_lts_2022.09_license_2022-12-27T083512.jwt
copy Dockerfile en install.sh naar ~/docker/  
```
https://www.cyberciti.biz/faq/explain-debian_frontend-apt-get-variable-for-ubuntu-debian/


4. install ubuntu 20.04 on the docker see 21:10 2e video  
```
sudo -i
sudo snap install docker 
cd ~/docker/
sudo docker pull ubuntu
sudo docker pull ubuntu:20.04
```
5. To this ubuntu we refer to in the first lines of the Dockerfile: FROM ubuntu:20.04  
  - For root password for ubuntu:
   https://linuxconfig.org/default-root-password-on-ubuntu-18-04-bionic-beaver-linux

6. docker build  


### install.sh  
```
#!/bin/bash
#

#cd /tmp
#mkdir /tmp/install
cd /tmp/install
#tar -xvzf /tmp/dockeresp2021_1.tar

DEBIAN_FRONTEND=noninteractive apt-get -yq update
DEBIAN_FRONTEND=noninteractive apt-get -yq install apt-utils
DEBIAN_FRONTEND=noninteractive apt-get -yq install curl


DEBIAN_FRONTEND=noninteractive apt-get -yq install /tmp/install/aarch64-ubuntu-linux-16/basic/*
DEBIAN_FRONTEND=noninteractive apt-get -yq install /tmp/install/aarch64-ubuntu-linux-16/analytics/*
DEBIAN_FRONTEND=noninteractive apt-get -yq install /tmp/install/aarch64-ubuntu-linux-16/textanalytics/*
DEBIAN_FRONTEND=noninteractive apt-get -yq install /tmp/install/aarch64-ubuntu-linux-16/astore/*
DEBIAN_FRONTEND=noninteractive apt-get -yq install /tmp/install/aarch64-ubuntu-linux-16/gpu/*


cp /tmp/install/SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt /opt/sas/viya/home/SASEventStreamProcessingEngine/etc/license/SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt
chown sas:sas /opt/sas/viya/home/SASEventStreamProcessingEngine/etc/license/SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt


chown sas:sas /opt/sas/viya/home/SASEventStreamProcessingEngine/lib/* 

cd /
rm -r /tmp/install
```
- Dockerfile  
``` 
FROM ubuntu:20.04

ENV DFESP_CONFIG=/data
ENV ODBCINI=/data/odbc.ini
ENV MAS_M2PATH=/opt/sas/viya/home/SASFoundation/misc/embscoreeng/mas2py.py
ENV MAS_PYPATH=/usr/bin/python3.8
ENV BASE_PRIMARY_LICENSE="SAS"
ENV BASE_PRIMARY_LICENSE_LOC=/data/SASViyaV4_0B17MY_lts_2021.2_license_2022-01-20T103652.jwt
ENV DFESP_HOME=/opt/sas/viya/home/SASEventStreamProcessingEngine
ENV LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$DFESP_HOME/ssl/lib:$DFESP_HOME/lib:/opt/sas/viya/home/SASFoundation/sasexe:/usr/lib"
ENV PATH=$PATH:$DFESP_HOME/bin
ENV SAS_VIYA_ENABLED=false

WORKDIR /tmp/install
ADD dockeresp2021_1.tar .
ADD install.sh .
RUN chmod u+x ./install.sh & /bin/bash ./install.sh
#ENTRYPOINT ["/bin/bash", "/install.sh"]
USER sas
CMD ["/opt/sas/viya/home/SASEventStreamProcessingEngine/bin/dfesp_xml_server"] 
```

4.2 build  
```
cd ~/docker/
docker build -t sasesp.2021.1:20220413.133100 .
```
4.3 add tags
```
docker images
docker image tag <id> sasesp.2021.1
docker image tag 6c3a02225d27  sasesp.2021.1
```
or:  
Since 1.10 release, you can now add multiple tags at once on build:
```
docker build -t name1:tag1 -t name1:tag2 -t name2 .
```

5. If something goes wrong you can clean up using: ( c2038f1eefae = image id):
```
docker image ls
docker image prune
docker stop f98c0c534e46
docker rm -f f98c0c534e46
docker stop caf0ca41cbea
docker rm caf0ca41cbea
docker image rm c2038f1eefae
docker image ls -a
docker ps -a
docker rm espserver.
```


6. copy esp-properties and jndi.properties from local install to docker  
```
cd ~/docker/
mkdir data
cp /opt/sas/viya/config/etc/SASEventStreamProcessingEngine/default/* data
```
6.2.  Edit esp-properties:  
```
connectors:
  excluded:
    mq: true
    tibrv: true
    sol: true
    tva: true
    pi: true
    tdata: true
    rmq: false
    sniffer: true
    mqtt: false
    pylon: true
    modbus: false
```
6. run docker  
```
docker run --name espserver -v ~/docker/data:/data .
docker run --name $NAME -v /home/ubuntu/docker/data:/data -d --restart unless-stopped -p 31415:31415 -p 31416:31416 --tty --group-add dialout --device=/dev/ttyS0:/dev/ttyS0 --env SETINIT_TEXT_ENC=$LICENSECONTENT --env DFESP_CONFIG=/data sasesp.2021.1
docker exec  -itu root espserver /bin/bash
curl http://baspi:31415/SASESP
```


### Extra:
To start docker with ubuntu user:

https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket

**the groupadd command creates a new group.
```
sudo groupadd docker
```
The command ‘usermod‘ is used to modify or change any attributes of a already created user account via command line.
   -a = To add anyone of the group to a secondary group.  
   -G = To add a supplementary groups.  
 $USER is current user  
``` 
sudo usermod -aG docker $USER
```
To start docker  using the esp.sh script of Alfredo:  /home/ubuntu/scripts  


### troubleshooting:
1. docker error - 'name is already in use by container'  
step 1:(it lists docker container with its name)
docker ps -a
step 2:
docker rm name_of_the_docker_container
stop is ctrl-c

 
## test on BASPi docker
```
cd ~/docker
./esp.sh start
docker exec  -itu root espserver /bin/bash 
./esp.sh shellAsRoot

cd /data
```
-  check if projects are running  
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects"
```
-  load project and run
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projectResults?windows=ParseMessage" -post  "file://lijster.xml"  
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projectResults" -post  "file://lijster.xml"  
dfesp_xml_client -url "http://baspi:31415/SASESP/projects/connectlijster" -put "file://connectlijster.xml"
dfesp_xml_client -url "http://baspi:31415/SASESP/projects/lijster" -put  "file://lijster.xml"
```

-  start loaded project
```
dfesp_xml_client -url "http://baspi:31415/SASESP/projects/connectlijster/state?value=running" -put
dfesp_xml_client -url "http://baspi:31415/SASESP/projects/lijster/state?value=running" -put
```
- check count
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/windows?count=true"
```
- view events:
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/events?windowFilter=eq(type,'window-aggregate')"
```
-  stop a project
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/lijster/state?value=stopped" -put
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/connectlijster/state?value=stopped" -put
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/connectlijster/state?value=started" -put
```
- delete project
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/lijster" -delete
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/connectlijster" -delete
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/projects/trades" -delete
```
- stop server
```
dfesp_xml_client -url "http://baspi:31415/eventStreamProcessing/v1/server/state?value=stopped" -put  
```
- restart baspi
To stop all projects, run the following command:
```
dfesp_xml_client -url "http://host :port/eventStreamProcessing/v1/stoppedProjects/project" -post
  sudo shutdown -r now
```

## Monitor the project  

View counts
http://baspi.localdomain:31415/eventStreamProcessing/v1/windows?count=true

View Stats
http://baspi.localdomain:31415/eventStreamProcessing/v1/projectStats?memory=true

View projects
http://baspi.localdomain:31415/eventStreamProcessing/v1/projects



--- 

## Reading thermometer data on BasPi:
https://cdn.en.papouch.com/data/user-content/old_eshop/files/TM/tm_en.pdf
This will set the baud rate to 9600, 8 bits, 1 stop bit, no parity:
sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb  
cat /dev/ttyS0

Test file does not contain a key column, so use autogen, set key as last column in output schema, and  use following properties:
```
        <property name="csvfielddelimiter"><![CDATA[,]]></property>  
        <property name="noautogenfield"><![CDATA[false]]></property>  
        <property name="addcsvopcode"><![CDATA[true]]></property>  
        <property name="addcsvflags"><![CDATA[normal]]></property>  
        <property name="fsname"><![CDATA[/mnt/data/input/lijster/testdata.csv]]></property>  
```

A CSV publisher converts each line into an event. The first two values of each
line are expected to be an opcode flag and an event flag. Use the addcsvopcode
and addcsvflags parameters when any line in the CSV file does not include an
opcode and event flag.

---

## Add slimme meter http://www.gejanssen.com/howto/Slimme-meter-uitlezen/  
### Slimme meter
```
sudo apt update
sudo apt install cu
cu -l /dev/ttyUSB0 -s 115200 --parity=none -E q &
```

To close cu, type “q.”, press Enter and wait a few seconds. The programm closes with the message Disconnected..

```
stty -F /dev/ttyUSB0 raw 115200 evenp
cat /dev/ttyUSB0
```

### P1 Telegram

| Value                                                                                                                 | OBIS reference  |
| --------------------------------------------------------------------------------------------------------------------- | --------------- |
| Version information for P1 output DSMR                                                                                | 1-3:0.2.8       |
| Date-time stamp of the P1 message                                                                                     | 0-0:1.0.0       |
| Equipment identifier                                                                                                  | 0-0:96.1.1.255  |
| Meter Reading electricity delivered to client (low tariff) in 0,001 kWh                                               | 1-0:1.8.1.255   |
| Meter Reading electricity delivered to client (normal tariff) in 0,001 kWh                                            | 1-0:1.8.2.255   |
| Meter Reading electricity delivered by client (low tariff) in 0,001 kWh                                               | 1-0:2.8.1.255   |
| Meter Reading electricity delivered by client (normal tariff) in 0,001 kWh                                            | 1-0:2.8.2.255   |
| Tariff indicator electricity. The tariff indicator can be used to switch tariff                                       | 0-0:96.14.0.255 |
| dependent loads e.g boilers. This is responsibility of the P1 user                                                    | 1-0:1.7.0.255   |
| Actual electricity power delivered (+P) in 1 Watt resolution                                                          | 1-0:2.7.0.255   |
| Actual electricity power received (-P) in 1 Watt resolution                                                           | 0-0:96.7.21.255 |
| Number of power failures in any phases                                                                                | 0-0:96.7. 9.255 |
| Number of long power failures in any phases                                                                           | 1-0:99:97.0.255 |
| Power failure event log                                                                                               | 1-0:32.32.0.255 |
| Number of voltage sags in phase L1                                                                                    | 1-0:52.32.0.255 |
| Number of voltage sags in phase L2                                                                                    | 1-0:72.32.0.255 |
| Number of voltage sags in phase L3                                                                                    | 1-0:32.36.0.255 |
| Number of voltage swells in phase L1                                                                                  | 1-0:52.36.0.255 |
| Number of voltage swells in phase L2                                                                                  | 1-0:72.36.0.255 |
| Number of voltage swells in phase L3                                                                                  | 1-0:32.7.0.255  |
| Instantaneous voltage L1                                                                                              | 1-0:52.7.0.255  |
| Instantaneous voltage L2                                                                                              | 1-0:72.7.0.255  |
| Instantaneous voltage L3                                                                                              | 1-0:31.7.0.255  |
| Instantaneous current L1                                                                                              | 1-0:51.7.0.255  |
| Instantaneous current L2                                                                                              | 1-0:71.7.0.255  |
| Instantaneous current L3                                                                                              | 1-0:21.7.0.255  |
| Instantaneous active power L1 (+P)                                                                                    | 1-0:41.7.0.255  |
| Instantaneous active power L2 (+P)                                                                                    | 1-0:61.7.0.255  |
| Instantaneous active power L3 (+P)                                                                                    | 1-0:22.7.0.255  |
| Instantaneous active power L1 (-P)                                                                                    | 1-0:42.7.0.255  |
| Instantaneous active power L2 (-P)                                                                                    | 1-0:62.7.0.255  |
| Instantaneous active power L3 (-P)                                                                                    | 1-0:62.7.0.255  |
| Text message max 1024 characters.                                                                                     | 0-0:96.13.0.255 |
| Device-Type                                                                                                           | 0-1:24.1.0.255  |
| Equipment identifier                                                                                                  | 0-1:96.1.0.255  |
| Last 5-minute value (temperature converted), gas delivered to client in m3, including decimal values and capture time | 0-1:24.2.1.255  |


---

# Data processing and reporting
1. real time power every second --> ESP connect
2. daily electricity consumption per hour (aggregated data) store  5 min data for gas, electricity and temp. --> SAS VA

---

## ESP Connect
- add graphics using react or c3js
Recorded session by Rik de Ruiter 30-9-2021
https://sasoffice365-my.sharepoint.com/personal/alfredo_iglesias_rey_sas_com/_layouts/15/stream.aspx?id=%2Fpersonal%2Falfredo%5Figlesias%5Frey%5Fsas%5Fcom%2FDocuments%2FRecordings%2FCommunity%20van%20ESP%2D20210930%5F131022%2DMeeting%20Recording%2Emp4&ga=1


### Use ESP connect

- Use powershell, http server: then nodejs  
https://github.com/sassoftware/esp-connect

PS C:\files\SourceTree\esp-connect> http-server --port 33000  
http://localhost:33000/html/modelviewer.html#
http://baspi.localdomain:31415


- OR use Visual Studio code
See 
- connect and load project in esp connect
_esp.connect(server,{ready:ready,error:error},{model:{name:_project,url:url},overwrite:true,force:_esp.getParm("force",true)});  
only connect and not load   (use project already running)
_esp.connect(server,{ready:ready,error:error},{model:null,overwrite:true,force:_esp.getParm("force",true)});  



- Swagger:
https://go.documentation.sas.com/doc/en/espcdc/6.1/espxmllayer/p111ycfjon4sran1a72zunszhq5x.htm
- [ ]
PS C:\files\SourceTree\myswagger\swagger-ui> cd ..
PS C:\files\SourceTree\myswagger> python -m http.server 55000
Serving HTTP on :: port 55000 (http://[::]:55000/) ...

---
# FTP

### use ftp server https://ubuntu.com/server/docs/service-ftp  in first version

https://www.cyberciti.biz/faq/howto-change-ssh-port-on-linux-or-unix-server/
Security, no write access and ssl

 Uncomment this to enable any form of FTP write command.
/etc/vsftpd.conf
#write_enable=YES

change home dir of ftp server
sudo usermod -d /home/sas/docker/data/output/lijster/ ftp 
check home dir
grep ftp /etc/passwd
restart ftp
sudo systemctl restart vsftpd.service


- Use view to autoload:
```
libname autol 'C:\SAS\Config\Lev1\AppData\SASVisualAnalytics\VisualAnalyticsAdministrator\AutoLoad';

filename p1_elec ftp 'elec_data.csv' cd='~/docker/data/output/lijster/'
   user='sas' 
   pass=sas 
   host='baspi.localdomain'
   recfm=v
   TERMSTR=LF
   PASSIVE  
   debug ;

/*filename p1_elec 'C:\temp\elec_data.csv';*/
proc delete data=autol.p1_elec (memtype=view);
run;
data autol.p1_elec 
/* Create a view */
/ view=autol.p1_elec
;   

filename p1_elec ftp 'elec_data.csv' cd='~/docker/data/output/lijster/'
   user='sas' 
   pass=sas 
   host='baspi.localdomain'
   recfm=v
   TERMSTR=LF
   PASSIVE  
   debug ;

   infile p1_elec dsd dlm=',' firstobs=2
/*obs=3     */
MISSOVER
lrecl=32767  
;
   format date datetime25.3;
   length p1_timestring $24;
   input 
   opcode $ state $  sensor_id period p1_timestring $ power electricity_low electricity_normal tariff;  
   date = Period/1;
run;

DATA view=autol.p1_elec; describe; run;
```
```
/* 
 * appserver_autoexec_usermods.sas
 *
 *    This autoexec file extends appserver_autoexec.sas.  Place your site-specific include 
 *    statements in this file.  
 *
 *    Do NOT modify the appserver_autoexec.sas file.  
 *    
 */
 
 libname autol 'C:\SAS\Config\Lev1\AppData\SASVisualAnalytics\VisualAnalyticsAdministrator\AutoLoad';

filename p1_elec ftp 'elec_data.csv' cd='~/docker/data/output/lijster/'
   user='sas' 
   pass=sas 
   host='baspi.localdomain'
   recfm=v
   TERMSTR=LF
   PASSIVE  
   debug ; 
```


- I reverse engineered the autoload process and found that this piece of code only looks at MEMTYPE=DATA. 
/opt/SAS/SASHome/SASVisualAnalyticsHighPerformanceConfiguration/7.3/Config/Deployment/Code/AutoLoad/include/GetTablesListFromLibrary.sas

I altered the use of the parameter TYPE, 2 places, to accept DATA and VIEW.
```
/*
 * GetTablesListFromLibrary()
 * 
 * Purpose: Given a libref, returns the list of members in that library.  Defaults to 
 *       tables list, unless type is specified.
 *
 * Parms:
 *       data=     The output data set to create.  If unspecified, defaults to work.tablelist.
 *      inlibref= The libref assigned to the source library to list.
 *    type=     The member type to list.  Defaults to DATA, but CATALOG can also be specified.
 */ 
%macro GetTablesListFromLibrary( data=WORK.tablelist, inlibref=, type=DATA, RENAME=NO );
   %setoption(off,notes);
   %if ("&inlibref." eq "" ) %then %do;
      %put ERROR: Source library must be specified with INLIBREF=;
      %return;
   %end;

   /* Redirect output */
   ods listing close;
   ods results off;
   ods output members=&DATA.; 

   /* Get Tables List */
   PROC DATASETS LIBRARY=&INLIBREF. MEMTYPE=(data view) nowarn;
   quit;
   %LET LISTCREATED=&SYSERR.;

   /* Restore output */
   ods results on;
   ods listing;

   %IF ( &RENAME. = YES ) %THEN %DO;
      %RenameColumnByIndex(data=&DATA., COLINDEX=4, NEWCOLNAME=FileSize);
      %RenameColumnByIndex(data=&DATA., COLINDEX=5, NEWCOLNAME=LastModified);
   %END;

   /* Add libref */
   data &data.;
      length fullref 
            $256;
      length name $128;
      format mdate NLDATMS27.;
      call missing(mdate);

      %IF "&LISTCREATED." eq "0" %THEN %DO;
         set &data.;
         fullref=kupcase("&inlibref.." || name);
         if kupcase(MemType) in ("DATA","VIEW");
		 if kupcase(MemType) in ("VIEW") then LastModified=datetime();
      %END;
      %ELSE %DO;
          call missing( fullref, name );
         delete;
      %END;
   run;

   %setoption(restore,notes);
%mend;
```

---

# Issues

### Issue with ttyS0 --> need to run after restart of BASPi
https://forum.arduino.cc/t/dev-ttyacm0-not-in-group-dialout/525321/3  
```
sudo systemctl stop serial-getty@ttyS0  
sudo systemctl disable serial-getty@ttyS0  
sudo chmod a+rw /dev/ttyS0  
ls -ltu /dev/ttyS0
```

###  regex Temp 

```
number(rgx('[^-]([0-9]*\.[0-9]*)C',$sensor_bericht,1))

<field name="temperatuur" type="double"/>       
  <expressions>  
    <expression name="temprgx"><![CDATA[[^-]([0-9]*\.[0-9]*)C]]></expression>  
  </expressions>    
  <functions>  
   <function name="temperatuur"><![CDATA[number(rgx(#temprgx,$sensor_bericht,1))]]></function>  
  </functions>	  
``` 
 

### fix issue with with TLS mixed content:

https://sirius.na.sas.com/Sirius/GSTS/ShowTrack.aspx?trknum=7613339506
https://go.documentation.sas.com/doc/nl/espcdc/v_021/dplyedge0phy0lax/p0oglqxbdzclxrn1cnq6k185zf8x.htm
Solved by using docker container for ESP Studio from Jan.

### change time on Pi   
timedatectl set-timezone Europe/Amsterdam 
 
 https://unix.stackexchange.com/questions/421354/convert-epoch-time-to-human-readable-in-libreoffice-calc
 
TST YYMMDDhhmmssX ASCII presentation of Time stamp with
Year, Month, Day, Hour, Minute, Second,
and an indication whether DST is active
(X=S) or DST is not active (X=W).

Why:
Epoch time is in seconds since 1/1/1970.
Calc internal time is in days since 12/30/1899.
So, to get a correct result in H3:

Get the correct number (last formula):

H3 = H2/(60*60*24) + ( Difference to 1/1/1970 since 12/30/1899 in days )
H3 = H2/86400      + ( DATE (1970,1,1) - DATE(1899,12,30) )
H3 = H2/86400      +   25569
But the epoch value you are giving is too big, it is three zeros bigger than it should. Should be 1517335200 instead of 1517335200000. It seems to be given in milliseconds. So, divide by 1000. With that change, the formula gives:

H3 = H2/1000/86400+25569  =  43130.75
Change the format of H3 to date and time (Format --> Cells --> Numbers --> Date --> Date and time) and you will see:

01/30/2018 18:00:00
in H3.

Of course, since Unix epoch time is always based on UTC (+0 meridian), the result above needs to be shifted as many hours as the local Time zone is distant from UTC. So, to get the local time, if the Time zone is Pacific standard time GMT-8, we need to add (-8) hours. The formula for H3 with the local time zone (-8) in H4 would be:

H3 = H2/1000/86400 + 25569 + H4/24 = 43130.416666
And presented as:

01/30/2018 10:00:00
if the format of H3 is set to such time format.

### issue with sudoers file:  

In the first terminal run the following command to get its PID.
echo $$
In the second terminal run
pkttyagent --process PID_FROM_STEP_1
In the first terminal, do whatever you need to do with pkexec.



### summer time correction (fixed)
https://blogs.sas.com/content/sasdummy/2015/04/16/how-to-convert-a-unix-datetime-to-a-sas-datetime/

 
###  stop project from web interface (skip)
###  opening model from web interface (skip)   
###  max power  (added)
###  usage instead of meter readings  (done)
###  Add thermostaat program  (done in VA Autoload)
###  symbolic link from sas web server to ESP connect  

#### Symbolic link ####
```
cd C:\SAS\Config\Lev1\Web\WebServer\htdocs
mklink /D esp-connect C:\files\SourceTree\esp-connect
ICACLS esp-connect /grant everyone:(OI)(CI)f /T
```
#### Edit config C:\SAS\Config\Lev1\Web\WebServer\conf\sas.conf ####
   disable conten-security-policy
#Header set Content-Security-Policy "default-src 'self' 'unsafe-inline' 'unsafe-eval'; img-src * data: blob:;  frame-src * blob: data: mailto:; child-src * blob: data: mailto:; font-src * data:;"
#### Use new links in VA: http://snlsea.emea.sas.com/esp-connect/examples/lijster/currentpower.html ###

### Open from outside the network  ###

  * Portforwarding in router (for T-Mobile Zyxel router Go to  192.168.1.1 - Netwerkinstelling - NAT )
    	
| \#  | Status | Servicenaam   | Oorspronkelijke IP | WAN-interface | Server IP-adres | Startpoort | Eindpoort | Vertaling startpoort | Vertaling eindpoort | Protocol | Wijzigen |     |
| --- | ------ | ------------- | ------------------ | ------------- | --------------- | ---------- | --------- | -------------------- | ------------------- | -------- | -------- | --- |
| 1   |        | EdgeESPSerfer |                    | ETH_Internet  | 192.168.1.90    | 31415      | 31416     | 31415                | 31416               | TCP      |          |     |
| 2   |        | ESPConnect    |                    | ETH_Internet  | 192.168.1.159   | 33000      | 33000     | 33000                | 33000               | ALL      |          |     |
| 3   |        | EgdeFTP       |                    | ETH_Internet  | 192.168.1.90    | 31422      | 31422     | 31422                | 31422               | TCP      |


- Add IP address from outside world to windows host file: 85.144.57.74		baspi.localdomain	  #baspi port fwd

- Open windows firewall: 

| Name                     | Group | Profile | Enabled | Action | Override | Program | Local Address | Remote Address | Protocol | Local Port   | Remote Port | Authorized Users | Authorized Computers | Authorized Local Principals | Local User Owner | Application Package |
| ------------------------ | ----- | ------- | ------- | ------ | -------- | ------- | ------------- | -------------- | -------- | ------------ | ----------- | ---------------- | -------------------- | --------------------------- | ---------------- | ------------------- |
| Allow ESP Server on Edge |       | All     | Yes     | Allow  | No       | Any     | Any           | Any            | TCP      | 31415, 31416 | Any         | Any              | Any                  | Any                         | Any              | Any                 |
| Allow ESP Connect        |       | All     | Yes     | Allow  | No       | Any     | Any           | Any            | TCP      | 33000        | Any         | Any              | Any                  | Any                         | Any              | Any                 |

- Use following fiveserver.config.js :
```
module.exports = {
    port: 33000,
    host: '192.168.1.159',
    //  host: 'localhost',
    https: false
}
```
	- Use chrome on iphone with 85.144.57.74:33000 as address and http://85.144.57.74:31415 as server.
	
	
	http://localhost:33000/examples/lijster/currentpower.html?server=http://baspi.localdomain:31415
	
	Add BASPi ftp port to portforwarding
	

### Archiving of data ###

  - Change code with view to a new data load program which accepts wild cards "C:\SAS\Config\Lev1\SASApp\SASEnvironment\SASCode\Jobs\load_p1_t_to_autoload.sas" 
  call this code from 
  "C:\SAS\Config\Lev1\Applications\SASVisualAnalytics\VisualAnalyticsAdministrator\AutoLoad.sas"
- cleanup of old data
  
cp p1_t.csv p1_t_$(date +%Y%m%d%H%M%S).csv
echo "sensor_id,period,temperatuur,power,power_max,electricity_low,electricity_low_usage,electricity_normal,electricity_normal_usage,tar iff,gas,gas_usage" > p1_t.csv
put code in shell script
put script in /etc/cron.monthly/

###  Implement secure ftp  ###
sshd_config  
PubkeyAuthentication yes  

https://superuser.com/questions/1647896/putty-key-format-too-new-when-using-ppk-file-for-putty-ssh-key-authentication  
- Install putty  
- Generate private key and public key (256 EdDSA Choose key, Parameters for saving key files, PPK file version 2) 
- No Passphrase SSH does not have an option to send the passphrase on the command line.
 
- save public key in ~/.ssh
ssh-keygen -i -f baspi8.pub >>  ~/.ssh/authorized_keys
use code 
test via windows prompt:
plink baspi.localdomain -i "C:\files\.ssh\baspi8.ppk"

```
filename  p1_t  SFTP   'p1_t.csv'  
            host="baspi.localdomain"
            wait_milliseconds=5000
            CD= "docker/data/output/lijster/"
              user="sas"
            options= "-v" debug
            optionsx="-i C:\files\.ssh\baspi8.ppk -v" 
            ;
```			
ssh authentication > only key!  		
https://learning.oreilly.com/library/view/ssh-the-secure/0596008953/ch05s04.html

```
   PubKeyAuthentication yes
   PasswordAuthentication no	
	Port 31422
sudo service ssh restart
```	

```	
sudo iptables -t nat -A PREROUTING -p tcp -d anywhere --dport 31422 -j DNAT --to-destination 192.168.1.90:31422	
sudo iptables -A FORWARD -p tcp -d 192.168.1.90 --dport 31422 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A POSTROUTING -t nat -p tcp -m tcp -s 192.168.1.90 --sport 31422 -j SNAT --to-source 85.144.57.74		
```	
	
https://www.transip.nl/knowledgebase/artikel/1937-uncomplicated-firewall-ufw-in-ubuntu/

```	
sudo ufw allow 31415:31416/tcp
sudo ufw allow 31422/tcp	
netstat -tulnp | grep "LISTEN"
sudo lsof -i tcp:31422
sudo netstat -lnp | grep ':31422 '

```
admin prompt:
```
Set Key="C:\files\.ssh\baspi82.ppk"
icacls "C:\files\.ssh\baspi82.ppk" /setowner "europe\snlsea"
Icacls %Key% /c /t /Inheritance:d
Icacls %Key% /c /t /Grant europe\snlsea:F
Icacls %Key%  /c /t /Remove Administrator BUILTIN\Administrators BUILTIN Everyone System Users
Icacls %Key%
set "Key="
```
- baspi83.ppk is a copy of the key file but owned by sasdemo, for batch processing
https://serverfault.com/questions/854208/ssh-suddenly-returning-invalid-format  
convert key file using keygen  

```
sftp -P 31422 -i "C:\files\.ssh\baspi82.ppk" sas@baspi.localdomain
```
### Add auto recovery from power failure and add Weekly restart to prevent memory issues

####  Add a startup ESP model to esp-properties.yml
https://documentation.sas.com/doc/en/espcdc/v_030/espxmllayer/p1r993p8bj4upxn1otx3e9ay5fcr.htm
esp-properties.yml check indentation and path
  model: "file:///data/lijster.xml"   Specify a file that contains XML code for a model to run when the ESP server starts 

#### create startup script: startup  
 https://smallbusiness.chron.com/run-command-startup-linux-27796.html 
Put a script containing the command in your /etc directory. 
Create a script such as "startup.sh" using your favorite text editor.
 Save the file in your /etc/init.d/ directory. C
 hange the permissions of the script (to make it executable) by typing "chmod +x /etc/init.d/mystartup.sh".

script is located in /home/sas/docker/startup  

/home/sas/docker/startup
```
[hash]!/bin/sh*
set -e

[hash] at startup  
sudo systemctl stop serial-getty@ttyS0  
sudo systemctl disable serial-getty@ttyS0  
sudo systemctl mask serial-getty@ttyS0.service
sudo chmod a+rw /dev/ttyS0  
sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb 
stty -F /dev/ttyUSB0 raw 115200 evenp
cd /home/sas/docker/data/output/lijster/
cp p1_t.csv p1_t_$(date +%Y%m%d%H%M%S).csv
echo "sensor_id,period,temperatuur,power,power_max,electricity_low,electricity_low_usage,electricity_normal,electricity_normal_usage,tar iff,gas,gas_usage" > p1_t.csv
cd /home/sas/docker
touch ./testje.txt
./esp.sh restart
```

#### schedule restart  

```
sudo crontab -e
0 0 * * 7 /sbin/shutdown -r +5
```

#### enable startup script at restart
```
sudo vi /etc/systemd/system/startup.service
[Unit]
Description=Disable cdrom

[Service]
Type=oneshot
ExecStart=/bin/sh /home/sas/docker/startup

[Install]
WantedBy=multi-user.target

systemctl enable startup.service
```
---

# TODO  

## ssl esp server
## add mqtt

---

# TIPS 
### 1. Linux
```
sudo -s
sudo chown bas:bas deploy    
tar -xvzf mirrormgr-linux.tgz  
```
- for repairing linefeeds to linux style  
```
sed -i -e 's/\r$//' studio.sh  
```
### 2. docker
- for all images also stopped  
```
docker ps -a  
remove image  
docker rm studio  
```
- version ubuntu  
```
lsb_release -a  
scp [OPTION] [user@]SRC_HOST:]file1 [user@]DEST_HOST:]file2  
```

- when you get deamon not started!
```
$ sudo snap start docker
```

---

# At startup of raspberry PI (this is now automated)

sudo systemctl stop serial-getty@ttyS0  
sudo systemctl disable serial-getty@ttyS0  
sudo systemctl mask serial-getty@ttyS0.service
sudo chmod a+rw /dev/ttyS0  
sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb 
stty -F /dev/ttyUSB0 raw 115200 evenp

\# list settings
stty -F /dev/ttyUSB0 --all

cd docker
./esp.sh restart


---

# TIPS 
## 1. Linux
```
sudo -s
sudo chown bas:bas deploy    
tar -xvzf mirrormgr-linux.tgz  
```
- for repairing linefeeds to linux style  
```
sed -i -e 's/\r$//' studio.sh  
```
## 2. docker
- for all images also stopped  
```
docker ps -a  
remove image  
docker rm studio  
```
- version ubuntu  
```
lsb_release -a  
scp [OPTION] [user@]SRC_HOST:]file1 [user@]DEST_HOST:]file2  
```

- when you get deamon not started!
```
$ sudo snap start docker
```
---



---
# Documentation
https://www.cyberciti.biz/faq/howto-setting-rhel7-centos-7-static-ip-configuration/   
https://www.tecmint.com/install-kubernetes-cluster-on-centos-7/  
https://phoenixnap.com/kb/how-to-install-kubernetes-on-centos   
https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/   
https://docs.docker.com/registry/deploying/  
https://github.com/sassoftware/esp-kubernetes  
https://kubernetes.io/docs/reference/kubectl/cheatsheet/  
https://repulpmaster.unx.sas.com/v2/cdp-release-x64_oci_linux_2-docker-latest/

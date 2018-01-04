nagios-invadelabs
-----------------
Setup for Invade Labs Nagios Docker using https://hub.docker.com/r/jasonrivers/nagios/

Pull down container:
```
$ docker pull jasonrivers/nagios:latest
```

Create directories for persistent data:
```
$ mkdir -p /home/drew/nagios/etc/ /home/drew/nagios/var /home/drew/nagios/Custom-Nagios-Plugins
```

Run The container:
```
$ docker run --name nagios4  \
  -v /home/drew/nagios/etc/:/opt/nagios/etc/ \
  -v /home/drew/nagios/var:/opt/nagios/var/ \
  -v /home/drew/nagios/Custom-Nagios-Plugins:/opt/Custom-Nagios-Plugins \
  -p 0.0.0.0:8080:80 jasonrivers/nagios:latest
```

Default login username / password http://host:8080/ :
```
nagiosadmin / nagios
```

Start / stop
```
$ docker start nagios4
$ docker stop nagios4
```

Start script, validate config, reload config
```
$ docker exec -it nagios4 /bin/bash
root@2c9389484f5b:/ # docker start nagios4 /usr/local/bin/start_nagios
root@2c9389484f5b:/ # nagios -v /opt/nagios/etc/nagios.cfg
root@2c9389484f5b:/ # kill -HUP <nagios pid>
```

## Fedora / CentOS/ RHEL Client Configuration
```
sudo dnf install nrpe nagios-plugins-all
```

/etc/nagrios/nrpe.cfg
```
allowed_hosts=172.0.0.1/10,192.168.1.0/24,127.0.0.1,::1 # on docker host add bridge ip..
include=/etc/nagios/nrpe_local.cfg
```

Create local config files
```
[drew@drew-serv nrpe.d]$ cat nrpe_local.cfg
######################################
# Do any local nrpe configuration here
######################################
command[check_users]=/usr/lib64/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib64/nagios/plugins/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/hda1
command[check_zombie_procs]=/usr/lib64/nagios/plugins/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/lib64/nagios/plugins/check_procs -w 200 -c 250
command[check_sda1]=/usr/lib64/nagios/plugins/check_disk -w 20% -c 10% -p /dev/sda1
```

Enable / Restart daemon
```
$ sudo systemctl enable nrpe
$ sudo systemctl start nrpe
```

##  Debian / Ubuntu Client Configuration
```
sudo apt-get install nagios-nrpe-server monitoring-plugins
```
Set allowed hosts and include local /etc/nagrios/nrpe.cfg
```
allowed_hosts=172.0.0.1/10,192.168.1.0/24,127.0.0.1,::1 # on docker host add bridge ip..
include=/etc/nagios/nrpe_local.cfg
```

Restart daemon
```
sudo service nagios-nrpe-server restart
```

Create local config
```
drew@drew-piv3:/etc/nagios $ cat nrpe_local.cfg
######################################
# Do any local nrpe configuration here
######################################
command[check_users]=/usr/lib/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib/nagios/plugins/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /dev/hda1
command[check_zombie_procs]=/usr/lib/nagios/plugins/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/lib/nagios/plugins/check_procs -w 150 -c 200
command[check_sda1]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /dev/mmcblk0p2
```

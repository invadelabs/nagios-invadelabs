nagios-invadelabs
-----------------
Setup for Invade Labs Nagios Docker using [jasonrivers/nagios](https://hub.docker.com/r/jasonrivers/nagios/)

*2018.05.28* - now using ansible to manage docker container life cycle in [invadelabs/ansible-invadelabs](https://github.com/invadelabs/ansible-invadelabs/blob/5c62a81a3bc845ff1d5bd4e1da790555b5dc21d9/local.yml#L236-L261).

Pull down container:
```
$ docker pull jasonrivers/nagios:latest
```

Create directories for persistent data:
```
$ mkdir -p /srv/nagios4/{etc,var,Custom-Nagios-Plugins,nagiosgraph/var}
```

Run The container (handled via invadelabs/ansible-invadelabs:drew-serv.yml):
```
$ docker run --name nagios4  \
  -v /srv/nagios4/etc:/opt/nagios/etc/ \
  -v /srv/nagios4/var:/opt/nagios/var/ \
  -v /srv/nagios4/Custom-Nagios-Plugins:/opt/Custom-Nagios-Plugins \
  -v /srv/nagios4/nagiosgraph/var:/opt/nagiosgraph/var \
  -v /srv/nagios4/main.cf:/etc/postfix/main.cf \
  -v /srv/nagios4/nagios.conf:/etc/apache2/sites-available/nagios.conf \
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

Copy this repo and add it to nagios.cfg as an include_dir:
```
grep nagios-invadelabs nagios.cfg
cfg_dir=/opt/nagios/etc/nagios-invadelabs
```

Other changes to nagios.cfg:
```
$ cat nagios.cfg.diff-2018-05-27
830,831c830,831
< host_perfdata_file=/opt/nagios/var/perfdata-host.log
< service_perfdata_file=/opt/nagios/var/perfdata-service.log
---
> #host_perfdata_file=/opt/nagios/var/host-perfdata
> service_perfdata_file=/opt/nagios/var/perfdata.log
843c843
< host_perfdata_file_template=$LASTHOSTCHECK$||$HOSTNAME$||$HOSTOUTPUT$||$HOSTPERFDATA$
---
> host_perfdata_file_template=empty
853c853
< host_perfdata_file_mode=a
---
> host_perfdata_file_mode=w
864c864
< host_perfdata_file_processing_interval=10
---
> #host_perfdata_file_processing_interval=0
874c874
< host_perfdata_file_processing_command=process-host-perfdata-file
---
> #host_perfdata_file_processing_command=process-host-perfdata-file
1175,1176c1175,1176
< admin_email=drew@invadelabs.com
< admin_pager=drew@invadelabs.com
---
> admin_email=nagios@localhost
> admin_pager=pagenagios@localhost
```

cgi.cfg (show all results by default):
```
result_limit=0
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

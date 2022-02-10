# Overview

Here are a couple of ansible playbooks to help install OpenBMP from scratch, or 
to prepare an existing system (before 2.0.3) for an upgrade.

The official documenation is located here: https://www.openbmp.org/getting_started.html

# Installing a new system

Use the build-openbmp.yml playbook to bring up OpenBMP on a new, single VM. Make sure to update 
these variables for your systems. The play is meant to be run in two steps. The first step will 
install docker and docker-compose, as well as pull the current config files needed build.

```
        - code_dir: "~/git" # location to install docker files and git repo
        - OBMP_DATA_ROOT: "/var/openbmp" # root of the openbmp data
        - docker_users: ['openbmp'] # user to add to the dockers group
```

The play is meant to be run in two steps. The first step will
install docker and docker-compose, as well as pull the current config files needed build.

```
 ansible-playbook -K -i hosts build-openbmp.yml

```

Copy the docker-compose_current.yml (that it just pulled from github) to docker-compose.yml 
and adjust for your environment. When it is ready, run the same play with the --tags=newbuild
to build the containers

```

 ansible-playbook -K -i hosts build-openbmp.yml --tags=newbuild

```

# Upgrading an older system
The upgrade process to go to 2.0.3 is documented here: https://www.openbmp.org/release_notes/2.0.3.html

The docker-compose.yml.pre_2.0.3 playbook helps to automate that process. Note that this will basically
remove all the data.

Update the ansible variables for your existing installation.

```
        - code_dir: "~/git" # location to install docker files and git repo
        - OBMP_DATA_ROOT: "/var/openbmp" # root of the openbmp data
        - docker_users: ['openbmp']
```


Run the playbook:

```

openbmp@foo:~/obmp_utils$ ansible-playbook -K -i hosts upgrade-pre_2.0.3.yml
BECOME password: 

PLAY [openbmp] ******************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [openbmp]

TASK [Confirm important file deletion] ******************************************************************************************
[Confirm important file deletion]
  This playbook will remove all data from a previous openbmp data created in /var/openbmp 
  and remove all existing docker containers, and prepare for a new 2.0.3+ install.
  https://www.openbmp.org/release_notes/2.0.3.html
  Do you want to continue? [yes/no]:
ok: [openbmp]

TASK [Confirm you have a backup config] *****************************************************************************************
[Confirm you have a backup config]
   Did you backup everything you need last chance [yes/no]:
ok: [openbmp]

TASK [stop the play] ************************************************************************************************************
skipping: [openbmp]

TASK [backup existing docker-compose.yml if there is a file there] **************************************************************
ok: [openbmp]

TASK [shutdwn and remove the existing containers, https://www.openbmp.org/release_notes/2.0.3.html] *****************************
changed: [openbmp]

TASK [remove the the /config/do_not_init_db file so the DB and be rebuilt] ******************************************************
ok: [openbmp]

TASK [remove old postgress data] ************************************************************************************************
[WARNING]: Consider using the file module with state=absent rather than running 'rm'.  If you need to use command because file
is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this
message.
changed: [openbmp]

TASK [remove old postgress data] ************************************************************************************************
changed: [openbmp]

TASK [remove old kafka data] ****************************************************************************************************
changed: [openbmp]

TASK [remove old zookeeper data] ************************************************************************************************
changed: [openbmp]

TASK [remove old zookeeper log] *************************************************************************************************
changed: [openbmp]

TASK [remove old grafana grafana/dashboards/] ***********************************************************************************
changed: [openbmp]

TASK [remove old grafana grafana/plugins] ***************************************************************************************
changed: [openbmp]

TASK [remove old grafana grafana/alerting] **************************************************************************************
changed: [openbmp]

TASK [remove old grafana grafana/provisioning/] *********************************************************************************
changed: [openbmp]

TASK [remove old grafana grafana/grafana.db] ************************************************************************************
changed: [openbmp]

TASK [openbmp : setup working directory] ****************************************************************************************
ok: [openbmp]

TASK [openbmp : backup existing docker-compose.yml if there is a file there] ****************************************************
ok: [openbmp]

TASK [openbmp : download docker-compose file to docker-compose_current.yml (so we we don't splash an existing config)] **********
ok: [openbmp]

TASK [openbmp : git clone github.com/OpenBMP/obmp-grafana.git] ******************************************************************
ok: [openbmp]

TASK [openbmp : ensure obmp_data_root exists. Make note of config guidance] *****************************************************
changed: [openbmp]

TASK [openbmp : directory warning] **********************************************************************************************
ok: [openbmp] => {
    "msg": " Remeber, you might want to create the directories by hand with high performance disks etc."
}

TASK [openbmp : create sub directories] *****************************************************************************************
ok: [openbmp] => (item=config)
ok: [openbmp] => (item=kafka-data)
ok: [openbmp] => (item=zk-data)
ok: [openbmp] => (item=zk-log)
ok: [openbmp] => (item=postgres)
ok: [openbmp] => (item=postgres/data)
ok: [openbmp] => (item=postgres/ts)
ok: [openbmp] => (item=grafana)

TASK [openbmp : copy grafana graphs into OBMP_DATA_ROOT/grafana/] ***************************************************************
changed: [openbmp] => (item=dashboards)
changed: [openbmp] => (item=provisioning)

TASK [Update the new docker-compose.yml file with your settings, and then re-run with --tags=newdbuild] *************************
ok: [openbmp] => {
    "msg": "copy the ~/git/docker-compose_current.yml to docker-compose.yml, and edit as needed.  Once you are ready, re-run this play with the --tags=newbuild to rebuild"
}

PLAY RECAP **********************************************************************************************************************
openbmp                    : ok=25   changed=13   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  

```


After the playbook completes, the containers should be gone

```
openbmp@openbmp:~/git$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

And the OBMP_DATA_ROOT should be empty

```

root@openbmp:/var/openbmp# tree .
.
├── config
│   └── obmp-psql.yml
├── grafana
│   ├── csv
│   └── png
├── kafka-data
├── postgres
│   ├── data
│   └── ts
├── zk-data
└── zk-log


```

And a new copy of gitrepo should be in the code_dir directory. Create a new docker-compose.yml based on docker-compose_current.yml file. Once complete, rebuild the containers


```

openbmp@foo:~/obmp_utils$ ansible-playbook -K -i hosts upgrade-pre_2.0.3.yml --tags=newbuild
BECOME password: 

PLAY [openbmp] ******************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [openbmp]

TASK [build the containers] *****************************************************************************************************
changed: [openbmp]

PLAY RECAP **********************************************************************************************************************
openbmp                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

This should recreate the containers:

```

openbmp@openbmp:~/git$ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS          PORTS                                       NAMES
66409e5b6f61   confluentinc/cp-kafka:7.0.1       "/etc/confluent/dock…"   25 seconds ago   Up 22 seconds   0.0.0.0:9092->9092/tcp, :::9092->9092/tcp   obmp-kafka
b1d45e284df1   confluentinc/cp-zookeeper:7.0.1   "/etc/confluent/dock…"   32 seconds ago   Up 25 seconds   2181/tcp, 2888/tcp, 3888/tcp                obmp-zookeeper
e76072d60aa2   openbmp/collector:2.0.3           "/usr/sbin/run"          32 seconds ago   Up 26 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   obmp-collector
56fa40a63f46   grafana/grafana:8.3.4             "/run.sh"                32 seconds ago   Up 25 seconds   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   obmp-grafana
463610859dab   openbmp/postgres:2.0.3            "docker-entrypoint.s…"   32 seconds ago   Up 25 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   obmp-psql
ed86cf46a83b   openbmp/psql-app:2.0.3            "/usr/sbin/run"          32 seconds ago   Up 6 seconds    0.0.0.0:9005->9005/tcp, :::9005->9005/tcp   obmp-psql-app



```




# Docker

## Run with `docker-compose`

A simple `docker-compose.yml`:


```bash
version: '2'

services:
  pd1:
    image: pingcap/pd
    ports:
      - "1234"
      - "9090"
      - "2379"
      - "2380"

    command:
      - --cluster-id=1 
      - --host=pd1 
      - --name=pd1 
      - --initial-cluster=pd1=http://pd1:2380,pd2=http://pd2:2380,pd3=http://pd3:2380

    privileged: true

  pd2:
    image: pingcap/pd
    ports:
      - "1234"
      - "9090"
      - "2379"
      - "2380"

    command:
      - --cluster-id=1 
      - --host=pd2
      - --name=pd2 
      - --initial-cluster=pd1=http://pd1:2380,pd2=http://pd2:2380,pd3=http://pd3:2380

    privileged: true

  pd3:
    image: pingcap/pd
    ports:
      - "1234"
      - "9090"
      - "2379"
      - "2380"

    command:
      - --cluster-id=1 
      - --host=pd3
      - --name=pd3 
      - --initial-cluster=pd1=http://pd1:2380,pd2=http://pd2:2380,pd3=http://pd3:2380 

    privileged: true

  tikv1:
    image: pingcap/tikv
    ports:
      - "20160"

    command:
      - --addr=0.0.0.0:20160
      - --advertise-addr=tikv1:20160
      - --cluster-id=1
      - --dsn=raftkv
      - --store=/var/tikv
      - --pd=pd1:2379,pd2:2379,pd3:2379

    depends_on:
      - "pd1"
      - "pd2"
      - "pd3"

    entrypoint: /tikv-server

    privileged: true

  tikv2:
    image: pingcap/tikv
    ports:
      - "20160"

    command:
      - --addr=0.0.0.0:20160
      - --advertise-addr=tikv2:20160
      - --cluster-id=1
      - --dsn=raftkv
      - --store=/var/tikv
      - --pd=pd1:2379,pd2:2379,pd3:2379

    depends_on:
      - "pd1"
      - "pd2"
      - "pd3"

    entrypoint: /tikv-server

    privileged: true

  tikv3:
    image: pingcap/tikv
    ports:
      - "20160"

    command:
      - --addr=0.0.0.0:20160
      - --advertise-addr=tikv3:20160
      - --cluster-id=1
      - --dsn=raftkv
      - --store=/var/tikv
      - --pd=pd1:2379,pd2:2379,pd3:2379

    depends_on:
      - "pd1"
      - "pd2"
      - "pd3"

    entrypoint: /tikv-server

    privileged: true

  tidb:
    image: pingcap/tidb
    ports:
      - "4000"

    command:
      - --store=tikv 
      - --path=pd1:2379,pd2:2379,pd3:2379/pd?cluster=1
      - -L=warn

    depends_on:
      - "tikv1"
      - "tikv2"
      - "tikv3"

    privileged: true
```

+ Use `docker-compose up -d` to create and start the cluster. 
+ Use `docker-compose port tidb 4000` to print the TiDB host port. For example, if the output is `0.0.0.0:32966`, the TiDB host port is `32966`.
+ Use `mysql -h 127.0.0.1 -P 32966 -u root -D test` to connect to TiDB and enjoy it. 
+ Use `docker-compose down` to stop and remove the cluster.
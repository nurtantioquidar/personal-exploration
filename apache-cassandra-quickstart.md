# Apache Cassandra



### General Information

Free and open-source, distributed, wide column store, NoSQL database management system designed to handle large amounts of data across many commodity servers, providing high availability with no single point of failure.

### Setup

- Using docker
  Pull the Cassandra server (<u>cannot pull using latest</u>, then the version is have to be specified):

  ```shell
  docker pull datastax/dse-server:6.8.4
  ```

  Pull the Cassandra UI:

  ```shell
  docker pull datastax/dse-studio
  ```

  Checking installation:

  ```shell
  nakama@BA-00003848:~$ docker image ls | grep datastax
  datastax/dse-server   6.8.4               0a4f9051971c        9 days ago          1.39GB
  datastax/dse-studio   latest              216b55489d35        22 months ago       595MB
  ```

- Getting up and running
  Starting single node server:

  ```shell
  docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server:6.8.4
  ```

  `-e` = sets the environment variables and accept the licensing agreement
  `-d` = starts the container in the background

  Checking the container that running:

  ```shell
  nakama@BA-00003848:~$ docker ps -a | grep dse
  57f7872cd317        datastax/dse-server:6.8.4   "/entrypoint.sh dse …"   About a minute ago   Up About a minute           4040/tcp, 5598-5599/tcp, 7000-7001/tcp, 7077/tcp, 7080-7081/tcp, 7199/tcp, 8090/tcp, 8182/tcp, 8609/tcp, 8983-8984/tcp, 9042/tcp, 9103/tcp, 9160/tcp, 9999-10000/tcp, 18080/tcp   my-dse
  
  ```

  Extra commands:

  `docker exec -it my-dse bash`: this command will open the interactive shell for that particular process

  `docker logs my-dse`: will show you logs for that particular process

  `docker stop my-dse`: stop the running container

  `docker rm /my-dse`: remove the running container

  Tips:

  Never use the ***rm /my-dse\*** commands because you will need to maintain the database and different schema you will create. The best way is to stop the container after your use and start it again for the next use.

  After you start the Cassandra node by using the docker run command, you can check the status of the node using:

  ```shell
  nakama@BA-00003848:~$ docker exec -it my-dse nodetool status
  Datacenter: dc1
  ===============
  Status=Up/Down
  |/ State=Normal/Leaving/Joining/Moving/Stopped
  --  Address     Load       Owns (effective)  Host ID                               Token                                    Rack
  UN  172.17.0.2  123.09 KiB  100.0%            26a34102-517f-4f5b-9477-5b18c17e17bf  -5078212206661688989                     rack1
  ```

  The code “**`UN`**” meaning it is up and running normally. 

### Playground

- Starting `cql` shell:

  ```shell
  docker exec -it my-dse bash
  nakama@BA-00003848:~$ docker exec -it my-dse bash
  dse@57f7872cd317:~$ cqlsh
  Connected to Test Cluster at 127.0.0.1:9042.
  [cqlsh 6.8.0 | DSE 6.8.4 | CQL spec 3.4.5 | DSE protocol v2]
  Use HELP for help.
  cqlsh> HELP
  
  Shell command help topics:
  ==========================
  CAPTURE  CLS          COPY     EXIT    HELP   PAGING  SHOW    TIMING   UNICODE
  CLEAR    CONSISTENCY  EXECUTE  EXPAND  LOGIN  SERIAL  SOURCE  TRACING
  
  Full documentation for shell commands:
    https://docs.datastax.com/en/dse/6.8/cql/cql/cql_reference/cqlsh_commands/cqlshCommandsTOC.html
  
  CQL command help topics:
  ========================
  ALTER_KEYSPACE             CREATE_ROLE           GRANT               
  ALTER_MATERIALIZED_VIEW    CREATE_SEARCH_INDEX   INSERT              
  ALTER_ROLE                 CREATE_TABLE          LIST_PERMISSIONS    
  ALTER_SEARCH_INDEX_CONFIG  CREATE_TYPE           LIST_ROLES          
  ALTER_SEARCH_INDEX_SCHEMA  CREATE_USER           LIST_USERS          
  ALTER_TABLE                DELETE                REBUILD_SEARCH_INDEX
  ALTER_TYPE                 DROP_AGGREGATE        RELOAD_SEARCH_INDEX 
  ALTER_USER                 DROP_FUNCTION         RESTRICT            
  BATCH                      DROP_INDEX            RESTRICT_ROWS       
  COMMIT_SEARCH_INDEX        DROP_KEYSPACE         REVOKE              
  CREATE_AGGREGATE           DROP_MATIALIZED_VIEW  SELECT              
  CREATE_CUSTOM_INDEX        DROP_ROLE             TRUNCATE            
  CREATE_FUNCTION            DROP_SEARCH_INDEX     UNRESTRICT          
  CREATE_INDEX               DROP_TABLE            UNRESTRICT_ROWS     
  CREATE_KEYSPACE            DROP_TYPE             UPDATE              
  CREATE_MATERIALIZED_VIEW   DROP_USER             USE
  ```

- Starting Cassandra `UI`:

  ```shell
  nakama@BA-00003848:~$ docker ps -a | grep dse-studio
  b1c3401a7023        datastax/dse-studio         "/entrypoint.sh"         21 seconds ago      Up 20 seconds               0.0.0.0:9091->9091/tcp                                                                                                                                                            my-studio
  ```

- We are exposing port 9091 by using `-p = expose the port`.
  ![image-20200928061751701](/home/nakama/.config/Typora/typora-user-images/image-20200928061751701.png)

- So we can play around with "`Working with CQL v6.0.0`" then setup the connection to be like this:
  ![image-20200928062116961](/home/nakama/.config/Typora/typora-user-images/image-20200928062116961.png)
  Change the `host/IP` to our host (that setup previously) which is `my-dse`.

### Cheat sheets

- Available cheat sheets:

  ```shell
  ========= Status =========
  #Active containers
  $> docker ps
  #Container Utilization
  $> docker stats
  #Container Details
  $> docker inspect my-dse
  #NodeTool Status
  $> docker exec -it my-dse nodetool status
  ========== Logs ==========
  #Server Logs
  $> docker logs my-dse
  #System Out
  $> docker exec -it my-dse cat /var/log/cassandra/system.log
  #Studio Logs
  $> docker logs my-studio
  ==== Start/Stop/Remove ====
  #Start Container
  $> docker start my-dse
  #Stop Container
  $> docker stop my-dse
  #Remove Container
  $> docker remove my-dse
  ======= Additional =======
  #Contaier IPAddress
  &> docker inspect my-dse | grep IPAddress
  #CQL (Requires IPAddress from above)
  $> docker exec -it my-dse cqlsh [IPAddress]
  #Bash
  $> docker exec -it my-dse bash
  ```

  

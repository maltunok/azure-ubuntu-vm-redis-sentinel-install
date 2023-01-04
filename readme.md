# INSTALL REDIS

1. Connect to each server over SSH.
2. Run the following commands to install Redis.

curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update

sudo apt-get install -y redis-server

3. Activate the redis-server service to start on boot.
   systemctl enable redis-server

# SETUP THE MASTER VM

1. Connect to your master server over SSH.
2. Open the configuration file at /etc/redis/redis.conf with your favorite editor.

bind 0.0.0.0
requirepass Passw0rd!
masterauth Passw0rd!

3. Restart the redis-server service.
   sudo service redis-server restart

# SETUP REPLICA VMs

1. Connect to each replica server over SSH.
2. Open the configuration file at /etc/redis/redis.conf with your favorite edit.
3. Find the following configuration, uncomment and edit them as follows. Change <YOUR_PASSWORD> to the same password on master node. Change <MASTER_NODE_IP> to the IP address of the master node.

bind 0.0.0.0
requirepass Passw0rd!
masterauth Passw0rd!
replicaof <MASTER_NODE_IP> 6379

4. Restart the redis-server service.
   sudo service redis-server restart

# SETUP FIREWALL FOR REDIS

1. Connect to the master server and each replica server over SSH.
2. Run the following commands. Change <MASTER_NODE_IP> to the IP address of the master node. Change <REPLICA_1_NODE_IP> to the IP address of the replica-1 node. Change <REPLICA_2_NODE_IP> to the IP address of the replica-2 node.

sudo ufw allow from <MASTER_NODE_IP> to any port 6379 comment 'MASTER'
sudo ufw allow from <REPLICA_1_NODE_IP> to any port 6379 comment 'REPLICA-1'
sudo ufw allow from <REPLICA_2_NODE_IP> to any port 6379 comment 'REPLICA-2'

# CHECK THE MASTER-REPLICA SETUP

1. Connect to your master server.
2. Check the log of Redis Sentinel.
   tail -f /var/log/redis/redis-server.log

3. Here is an example result:
   704:M 01 Oct 2022 09:20:58.524 _ Loading RDB produced by version 7.0.5
   704:M 01 Oct 2022 09:20:58.524 _ RDB age 15 seconds
   704:M 01 Oct 2022 09:20:58.524 _ RDB memory usage when created 0.91 Mb
   704:M 01 Oct 2022 09:20:58.524 _ Done loading RDB, keys loaded: 1, keys expired: 0.
   704:M 01 Oct 2022 09:20:58.525 _ DB loaded from disk: 0.002 seconds
   704:M 01 Oct 2022 09:20:58.525 _ Ready to accept connections
   704:M 01 Oct 2022 09:20:59.353 _ Replica XXX:6379 asks for synchronization
   704:M 01 Oct 2022 09:20:59.353 _ Partial resynchronization request from XXX:6379 accepted. Sending 0 bytes of backlog starting from offset 2873.
   704:M 01 Oct 2022 09:20:59.435 _ Replica XXX:6379 asks for synchronization
   704:M 01 Oct 2022 09:20:59.435 _ Partial resynchronization request from XXX:6379 accepted. Sending 0 bytes of backlog starting from offset 2873.

4. Run Redis CLI to connect to the Redis server.
   redis-cli

5. Run AUTH command to authenticate. Change <YOUR_PASSWORD> to the same password on master node.
   AUTH Passw0rd!

6. Run INFO command to check the Redis replication information.
   INFO REPLICATION

7. Here is an example result. The output connected_slaves:2 shows that both replica nodes have successfully connected to the master node.

# Replication

role:master
connected_slaves:2
slave0:ip=XXX,port=6379,state=online,offset=14,lag=1
slave1:ip=XXX,port=6379,state=online,offset=14,lag=1
master_failover_state:no-failover
master_replid:43855ff334c072ee5a3f4db39b1b27b31f55a3a4
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14

8. Run SET command to set a key hello to world on the master node.
   SET hello devbase

9. Connect to your redis replica server.

10. Run Redis CLI to connect to the Redis server.
    redis-cli

11. Run AUTH command to authenticate. Change <YOUR_PASSWORD> to the same password on the master node.
    AUTH Passw0rd!

12. Run GET command to get the value of key hello. The result is world shows that the replication from master to replica works successfully.
    GET hello

# CONFIGURE REDIS SENTINEL

1. On all three servers, install Redis Sentinel as follows:
   sudo apt install redis-sentinel

2. Activate the redis-sentinel service to start on boot.
   systemctl enable redis-sentinel

3. Open the configuration file at /etc/redis/sentinel.conf with your favorite editor.

4. Find the line sentinel monitor mymaster 127.0.0.1 6379 2, and edit them as follows. Change <YOUR_PASSWORD> to the same password on master node. Change <MASTER_NODE_IP> to the IP address of the master node.

sentinel monitor mymaster <MASTER_NODE_IP> 6379 2
sentinel auth-pass mymaster Passw0rd!
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
protected-mode no

5. Restart the Redis Sentinel Service.
   sudo service redis-sentinel restart

# SETUP FIREWALL FOR SENTINEL

1. Connect to the master server and each replica server over SSH.

2. Run the following commands. Change <MASTER_NODE_IP> to the IP address of the master node. Change <REPLICA_1_NODE_IP> to the IP address of the replica-1 node. Change <REPLICA_2_NODE_IP> to the IP address of the replica-2 node.

sudo ufw allow from <MASTER_NODE_IP> to any port 26379 comment 'SENTINEL'
sudo ufw allow from <REPLICA_1_NODE_IP> to any port 26379 comment 'SENTINEL'
sudo ufw allow from <REPLICA_2_NODE_IP> to any port 26379 comment 'SENTINEL'

# CHECK THE SENTINEL SETUP

1. Connect to your master server over SSH.

2. Check the log of Redis Sentinel.
   tail -f /var/log/redis/redis-sentinel.log

3. Here is an example result:
   1049:X 01 Oct 2022 17:57:30.950 _ Removing the pid file.
   1049:X 01 Oct 2022 17:57:30.950 # Sentinel is now ready to exit, bye bye...
   5020:X 01 Oct 2022 17:57:30.993 _ Supervised by systemd. Please make sure you set appropriate values for TimeoutStartSec and TimeoutStopSec in your service unit.
   5020:X 01 Oct 2022 17:57:30.993 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   5020:X 01 Oct 2022 17:57:30.993 # Redis version=7.0.5, bits=64, commit=00000000, modified=0, pid=5020, just started
   5020:X 01 Oct 2022 17:57:30.993 # Configuration loaded
   5020:X 01 Oct 2022 17:57:30.994 _ monotonic clock: POSIX clock_gettime
   5020:X 01 Oct 2022 17:57:30.994 _ Running mode=sentinel, port=26379.
   5020:X 01 Oct 2022 17:57:30.995 # Sentinel ID is 8de12f567ebdfdfc8df229497e445667fd18f3e2
   5020:X 01 Oct 2022 17:57:30.995 # +monitor master mymaster 45.32.121.40 6379 quorum 2

4. Run Redis CLI to connect to the Redis Sentinel server.
   redis-cli -p 26379

5. Run INFO command to check the Redis replication information.
   INFO SENTINEL

6. Here is an example result.

# Sentinel

sentinel_masters:1
sentinel_tilt:0
sentinel_tilt_since_seconds:-1
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=<MASTER_NODE_IP>:6379,slaves=2,sentinels=3

# TEST THE FAILOVER

This section shows how to test the failover by stopping the Redis master node manually.

1. Connect to your master server.

2. Stop the Redis services on the master server.
   sudo service redis-sentinel stop
   sudo service redis-server stop

3. Open a new terminal and connect to a Redis replica server.

4. Check the log of Redis Sentinel.
   tail -f /var/log/redis/redis-sentinel.log

5. Here is an example result. The Sentinel starts a failover process and promotes a replica to be the new master.
   5222:X 01 Oct 2022 18:59:38.903 # +sdown sentinel e5b67f1e144dedf55cf71e60df52489debfdcb9d 45.32.121.40 26379 @ mymaster 45.32.121.40 6379
   5222:X 01 Oct 2022 18:59:39.836 # +sdown master mymaster XXX 6379
   5222:X 01 Oct 2022 18:59:39.958 _ Sentinel new configuration saved on disk
   5222:X 01 Oct 2022 18:59:39.958 # +new-epoch 3
   5222:X 01 Oct 2022 18:59:39.959 _ Sentinel new configuration saved on disk
   5222:X 01 Oct 2022 18:59:39.960 # +vote-for-leader 852870198de5b3ced3abd82b8f85cd4866c7921c 3
   5222:X 01 Oct 2022 18:59:40.975 # +odown master mymaster XXX 6379 #quorum 2/2
   5222:X 01 Oct 2022 18:59:40.976 # Next failover delay: I will not start a failover before Sat Oct 1 19:01:40 2022
   5222:X 01 Oct 2022 18:59:41.028 # +config-update-from sentinel 852870198de5b3ced3abd82b8f85cd4866c7921c XXX 26379 @ mymaster XXX 6379
   5222:X 01 Oct 2022 18:59:41.028 # +switch-master mymaster XXX 6379 XXX 6379
   5222:X 01 Oct 2022 18:59:41.028 _ +slave slave XXX:6379 XXX 6379 @ mymaster XXX 6379
   5222:X 01 Oct 2022 18:59:41.029 _ +slave slave XXX:6379 XXX 6379 @ mymaster XXX 6379
   5222:X 01 Oct 2022 18:59:41.030 \* Sentinel new configuration saved on disk
   5222:X 01 Oct 2022 18:59:46.128 # +sdown slave XXX:6379 XXX 6379 @ mymaster XXX 6379

6. Run Redis CLI to connect to the Redis Sentinel server.
   redis-cli -p 26379

7. Get the master address.
   SENTINEL get-master-addr-by-name mymaster

8. Here is an example result with the IP address and port of the master node.

1) "XXX"
2) "6379"

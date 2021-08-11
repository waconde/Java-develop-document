# centOS使用systemctl制作clickhouse服务
##  systemctl
```
[Unit]
Description=ClickHouse Server (analytic DBMS for big data)
Requires=network-online.target
After=network-online.target

[Service]
Type=simple
User=clickhouse
Group=clickhouse
Restart=always
RestartSec=30
RuntimeDirectory=clickhouse-server
ExecStart=/usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid
LimitCORE=infinity
LimitNOFILE=500000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_IPC_LOCK CAP_SYS_NICE

[Install]
WantedBy=multi-user.target
```
说明：
 
### Unit：
要求network-online.target服务启动。在network-online.target服务启动之后启动。 
### Service
User：启动用户
Group：启动用户所属的组
Restart： 重启
RestartSec： 重启时间
ExecStart： 重启命令
##  clickhouse数据库配置文件说明
重启命令/usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid指定了配置文件和pid文件

配置文件同级目录如下： 

```
[root@server-6 clickhouse-server]# ll
总用量 40
drwxr-xr-x 2 clickhouse clickhouse     6 3月   4 16:29 config.d
-rw-r--r-- 1 clickhouse clickhouse 20696 3月   4 16:35 config.xml
-rw-r--r-- 1 clickhouse clickhouse  3326 6月  19 12:37 metrika.xml
drwxr-xr-x 2 clickhouse clickhouse     6 3月   4 16:29 users.d
-rw-r--r-- 1 clickhouse clickhouse  8917 3月   4 16:35 users.xml
[root@server-6 clickhouse-server]#

```
config.xml内容引用metrika.xml和users.xml 两个配置文件

```
<yandex>
   <http_port>8123</http_port>
   <tcp_port>9000</tcp_port>
   ……
   <users_config>users.xml</users_config>
   
   <include_from>/etc/clickhouse-server/metrika.xml</include_from>
   
</yandex>
```

users.xml 

```
<yandex>
    <!-- Profiles of settings. -->
    <profiles>
      <default>
            <!-- Maximum memory usage for processing single query, in bytes. -->
            <max_memory_usage>100000000000</max_memory_usage>
           <max_memory_usage_for_all_queries>200000000000</max_memory_usage_for_all_queries>
           <!--<max_bytes_before_external_group_by>50000000000</max_bytes_before_external_group_by>-->

            <!-- Use cache of uncompressed blocks of data. Meaningful only for processing many of very short queries. -->
            <use_uncompressed_cache>0</use_uncompressed_cache>

            <!-- How to choose between replicas during distributed query processing.
                 random - choose random replica from set of replicas with minimum number of errors
                 nearest_hostname - from set of replicas with minimum number of errors, choose replica
                  with minimum number of different symbols between replica's hostname and local hostname
                  (Hamming distance).
                 in_order - first live replica is chosen in specified order.
                 first_or_random - if first replica one has higher number of errors, pick a random one from replicas with minimum number of errors.
            -->
            <load_balancing>random</load_balancing>
            <log_queries>1</log_queries>
            <distributed_product_mode>local</distributed_product_mode>
            <allow_experimental_data_skipping_indices>1</allow_experimental_data_skipping_indices>
            <max_execution_time>300</max_execution_time>
            <timeout_overflow_mode>throw</timeout_overflow_mode>
            <background_pool_size>32</background_pool_size>
            <skip_unavailable_shards>1</skip_unavailable_shards>
            <max_query_size>1048576</max_query_size>
        </default>
        
        <readonly>
            <!-- Maximum memory usage for processing single query, in bytes. -->
            <max_memory_usage>100000000000</max_memory_usage>
           <max_memory_usage_for_all_queries>200000000000</max_memory_usage_for_all_queries>
          <!--  <max_bytes_before_external_group_by>50000000000</max_bytes_before_external_group_by>-->

            <!-- Use cache of uncompressed blocks of data. Meaningful only for processing many of very short queries. -->
            <use_uncompressed_cache>0</use_uncompressed_cache>

            <!-- How to choose between replicas during distributed query processing.
                                  random - choose random replica from set of replicas with minimum number of errors
                 nearest_hostname - from set of replicas with minimum number of errors, choose replica
                  with minimum number of different symbols between replica's hostname and local hostname
                  (Hamming distance).
                 in_order - first live replica is chosen in specified order.
                 first_or_random - if first replica one has higher number of errors, pick a random one from replicas with minimum number of errors.
            -->
            <load_balancing>random</load_balancing>
            <log_queries>1</log_queries>
            <distributed_product_mode>local</distributed_product_mode>
            <allow_experimental_data_skipping_indices>1</allow_experimental_data_skipping_indices>
            <max_execution_time>300</max_execution_time>
            <timeout_overflow_mode>throw</timeout_overflow_mode>
            <background_pool_size>32</background_pool_size>
            <skip_unavailable_shards>1</skip_unavailable_shards>
            <readonly>2</readonly>
        </readonly>
    </profiles>
    
     <users>
        <!-- If user name was not specified, 'default' user is used. -->
        <default>
            <!-- Password could be specified in plaintext or in SHA256 (in hex format).

                 If you want to specify password in plaintext (not recommended), place it in 'password' element.
                 Example: <password>qwerty</password>.
                 Password could be empty.

                 If you want to specify SHA256, place it in 'password_sha256_hex' element.
                 Example: <password_sha256_hex>65e84be33532fb784c48129675f9eff3a682b27168c0ea744b2cf58ee02337c5</password_sha256_hex>
                 Restrictions of SHA256: impossibility to connect to ClickHouse using MySQL JS client (as of July 2019).

                 If you want to specify double SHA1, place it in 'password_double_sha1_hex' element.
                 Example: <password_double_sha1_hex>e395796d6546b1b65db9d665cd43f0e858dd4303</password_double_sha1_hex>

                 How to generate decent password:
                 Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
                 In first line will be password and in second - corresponding SHA256.

                 How to generate double SHA1:
                 Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | openssl dgst -sha1 -binary | openssl dgst -sha1
                 In first line will be password and in second - corresponding double SHA1.
            -->
            <password></password>

            <!-- List of networks with open access.

                 To open access from everywhere, specify:
                    <ip>::/0</ip>

                 To open access only from localhost, specify:
                    <ip>::1</ip>
                    <ip>127.0.0.1</ip>

                 Each element of list has one of the following forms:
                 <ip> IP-address or network mask. Examples: 213.180.204.3 or 10.0.0.1/8 or 10.0.0.1/255.255.255.0
                     2a02:6b8::3 or 2a02:6b8::3/64 or 2a02:6b8::3/ffff:ffff:ffff:ffff::.
                 <host> Hostname. Example: server01.yandex.ru.
                     To check access, DNS query is performed, and all received addresses compared to peer address.
                 <host_regexp> Regular expression for host names. Example, ^server\d\d-\d\d-\d\.yandex\.ru$
                     To check access, DNS PTR query is performed for peer address and then regexp is applied.
                     Then, for result of PTR query, another DNS query is performed and all received addresses compared to peer address.
                     Strongly recommended that regexp is ends with $
                 All results of DNS requests are cached till server restart.
            -->
            <networks incl="networks" replace="replace">
                <ip>::/0</ip>
            </networks>

            <!-- Settings profile for user. -->
            <profile>default</profile>

            <!-- Quota for user. -->
            <quota>default</quota>

            <!-- For testing the table filters -->
            <databases>
                <test>
                    <!-- Simple expression filter -->
                    <filtered_table1>
                        <filter>a = 1</filter>
                    </filtered_table1>

                    <!-- Complex expression filter -->
                    <filtered_table2>
                        <filter>a + b &lt; 1 or c - d &gt; 5</filter>
                    </filtered_table2>

                    <!-- Filter with ALIAS column -->
                    <filtered_table3>
                        <filter>c = 1</filter>
                    </filtered_table3>
                </test>
            </databases>
        </default>

        <!-- Example of user with readonly access. -->
        <!-- <readonly>
            <password></password>
            <networks incl="networks" replace="replace">
                <ip>::1</ip>
                <ip>127.0.0.1</ip>
            </networks>
            <profile>readonly</profile>
            <quota>default</quota>
        </readonly> -->
        <writer>
                <password>k18</password>
                <networks incl="networks" replace="replace">
                        <ip>::/0</ip>
                </networks>
                <profile>default</profile>
                <quota>default</quota>
        </writer>

    </users>

    <!-- Quotas. -->
    <quotas>
        <!-- Name of quota. -->
        <default>
            <!-- Limits for time interval. You could specify many intervals with different limits. -->
            <interval>
                <!-- Length of interval. -->
                <duration>3600</duration>

                <!-- No limits. Just calculate resource usage for time interval. -->
                <queries>0</queries>
                <errors>0</errors>
                <result_rows>0</result_rows>
                <read_rows>0</read_rows>
                <execution_time>0</execution_time>
            </interval>
        </default>
    </quotas>
   
</yandex>    
 ```


在配置文件/etc/clickhouse-server/config.xml中，有配置用户的文件
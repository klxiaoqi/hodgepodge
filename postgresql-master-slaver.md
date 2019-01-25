# 准备工作

---



| hostname    | ip            |           |
| ----------- | ------------- | --------- |
| anxinyun-m1 | 192.168.0.201 | pg-master |
| anxinyun-m2 | 192.168.0.206 | pg-slaver |



> 安装好 postgresql



# 配置

## 配置master

> 

> 修改 **postgresql.conf**



```bash
vi /etc/postgresql/9.6/main/postgresql.conf
> 
> listen_addresses = 'localhost,192.168.0.201'
> wal_level = 'hot_standby'
> # 同步级别，改为 本地同步
> synchronous_commit = local
> # 启用归档模式,修改archive_command选项
> archive_mode = on
> archive_command = 'cp %p /var/lib/postgresql/9.6/main/archive/%f'
> # 我们仅用2个服务器，主服务器和从服务器 
> max_wal_senders = 2
> wal_keep_segments = 10
> # 
> synchronous_standby_names = 'pgslave001'
```

> 保存退出

> 编辑 修改 pg_hba.conf

```bash
vi pg_hba.conf
>
> # Localhost
> st    replication     replica          127.0.0.1/32            md5
> # PostgreSQL Master IP address
> host    replication     replica          192.168.0.201/24            md5
> # PostgreSQL SLave IP address
> host    replication     replica          192.168.0.206/24            md5

```



> 在postgresql.conf文件中，归档模式已启用，因此我们需要为归档创建一个新目录。 创建一个新的归档目录，更改权限并将所有者更改为postgres用户

```bash
su - postgres
mkdir -p /var/lib/postgresql/9.6/main/archive/
chmod 700 /var/lib/postgresql/9.6/main/archive/
chown -R postgres:postgres /var/lib/postgresql/9.6/main/archive/
```

> 保存退出

> 重启 postgresql 服务 , 查看服务是否启动成功

```bash
systemctl restart postgresql

netstat -plntu
# 或
ps -ef | grep postgres

```

> 创建一个用户用于复制, 用户名：' **replica** '，密码:' **aqwe123 @** '

```bash
su - postgres
psql
postgres=# CREATE USER replica REPLICATION LOGIN ENCRYPTED PASSWORD 'aqwe123@';
# 查看用户
postgres=# \du

```



## 配置 slaver

> 停止 postgresql 服务

```bash
systemctl stop postgresql
```

> 编辑 **/etc/postgresql/9.6/main/postgresql.conf**

```bash
vi /etc/postgresql/9.6/main/postgresql.conf
> 
> listen_addresses = 'localhost,192.168.0.206'
> wal_level = hot_standby
> # 同步级别，改为 本地同步
> synchronous_commit = local
> # 我们仅用2个服务器，主服务器和从服务器 
> max_wal_senders = 2
> wal_keep_segments = 10
> # 
> synchronous_standby_names = 'pgslave001'
> 启用从服务器的hot_standby 
> hot_standby = on
```

> 保存并退出

## 复制数据从MASTER复制到SLAVE

> 登录SLAVE服务器并访问postgres用户

```bash
su - postgres
```

> 转到postgres数据目录' **main** '，并通过重命名目录名来备份

```bash
cd 9.6/
mv main main-bekup
```

> 使用**pg_basebackup**命令将主目录从MASTER服务器复制到SLAVE服务器

```bash
pg_basebackup -h 10.0.15.10 -U replica -D /var/lib/postgresql/9.6/main -P --xlog
Password:
```

> 数据传输完成后，转到主数据目录并创建一个新的**recovery.conf**文件

```bash
cd /var/lib/postgresql/9.6/main/
vim recovery.conf
```

> 复制粘贴以下配置

```
standby_mode = 'on'
primary_conninfo = 'host=192.168.0.201 port=5432 user=replica password=aqwe123@ application_name=pgslave001'
restore_command = 'cp /var/lib/postgresql/9.6/main/archive/%f %p'
trigger_file = '/tmp/postgresql.trigger.5432'
```

> 保存退出

> 修改权限

```
chmod 600 recovery.conf
```

> 启动 slaver  postgresql 服务, 查看服务

```bash
systemctl start postgresql
netstat -plntu
# 或
ps -ef | grep postgres
```

## 测试

> 登录 master postgres 用户

```bash
su - postgres
psql -c "select application_name, state, sync_priority, sync_state from pg_stat_replication;"
psql -x -c "select * from pg_stat_replication;"
```

![](/assets/pg_02.png)

> 再MASTER服务器创建一个新表**replica_test**  并插入测试数据

```bash
su - postgres
psql

CREATE TABLE replica_test (hakase varchar(100));
INSERT INTO replica_test VALUES ('howtoing.com');
INSERT INTO replica_test VALUES ('This is from Master');
INSERT INTO replica_test VALUES ('pg replication by hakase-labs');
```

![](/assets/pg_01.png)

> 登录 slaver 查询  ' **replica_test** '表中的数据

```bash
su - postgres
psql

select * from replica_test;
```

![](/assets/pg_03.png)

> **附加测试**

> 测试在SLAVE服务器上写插入语句

```bash
INSERT INTO replica_test VALUES ('this is SLAVE');
```

> 你将在SLAVE服务器上收到有关“ **无法执行INSERT** ”查询的错误消息



> 至此，在Ubuntu 16.04上安装和配置PostgreSQL 9.6与主从复制已经完成
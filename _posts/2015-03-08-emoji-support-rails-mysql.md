---
layout: post
title: 让 MySQL+Rails 项目支持表情
---

为了能够让MySQL支持Emoji的存储，需要对数据库的存储编码方式进行修改

###导出数据
假设数据库名为 youyushe，首先导出数据库数据

```
mysqldump --ignore-table=youyushe.schema_migrations -u root -p --no-create-info youyushe > ~/youyu_classroom_data.sql
```

###修改配置
MySQL 5.5.3以上已经支持了utf8mb4的字符集，所以如果你的MySQL是5.5.3以上，只需要在my.cnf文件中按照如下的配置就可以支持utf8mb4的字符集了。

```ruby
[client]
default-character-set = utf8mb4

[mysqld]
collation-server = utf8mb4_unicode_ci
character-set-server = utf8mb4
```
my.cnf文件一般存放在/etc/my.cnf或者/usr/local/etc/my.cnf中，如果没有的话，自己创建一个。

如果MySQL版本较低，请先升级MySQL的版本
Mac用户可以通过brew升级

```
brew update
brew upgrade mysql
```

Ubuntu用户可以通过apt升级

```
sudo apt-get update
sudo apt-get upgrade mysql
```

在修改完my.conf配置之后，重启mysql，检查字符集是否已经更改，除了character_set_system和character_set_filesystem之外，其他的字符集都需要变成utf8mb4类型。

```
mysql> show variables like 'char%';
+--------------------------+--------------------------------------------------------------------+
| Variable_name            | Value                                                              |
+--------------------------+--------------------------------------------------------------------+
| character_set_client     | utf8mb4                                                            |
| character_set_connection | utf8mb4                                                            |
| character_set_database   | utf8mb4                                                            |
| character_set_filesystem | binary                                                             |
| character_set_results    | utf8mb4                                                            |
| character_set_server     | utf8mb4                                                            |
| character_set_system     | utf8                                                               |
| character_sets_dir       | /usr/local/Cellar/percona-server/5.6.23-72.1/share/mysql/charsets/ |
+--------------------------+--------------------------------------------------------------------+
8 rows in set (0.00 sec)
```

Rails数据库配置文件中的encoding必须改为utf8mb4，需要修改application.yml，可以参考application.yml.example的改动

```
production:
  DB_ADAPTER: mysql2
  DB_ENCODING: utf8mb4
  DB_DATABASE: youyushe
  DB_USERNAME: root
  DB_PASSWORD:
  DB_HOST: localhost
  DB_PORT: 3306
```
### 重建数据库结构

```
bundle exec rake db:drop
bundle exec rake db:create
bundle exec rake db:migrate

```

### 修改数据并导入

修改dump数据的第10行，从utf8改为utf8mb4

```
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
```

导入数据

```
mysql -uroot youyushe < youyu_classroom_data.sql
```
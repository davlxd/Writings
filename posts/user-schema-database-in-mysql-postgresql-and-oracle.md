# User, Schema, Database in MySQL, PostgreSQL, and Oracle
2017-04-01

`User`, `Schema`, and `Database` are fundamental concepts for every RDBMS, but they are easy to forget unless you are dedicated DBA, since most of the time you only need this knowledge when you bootstrap a new project. This kind of situation has happened to me countless times, so in this post, I'm solving this problem once for all, and I hope it could be helpful for you as well.


## MySQL/MariaDB

MySQL is relatively simple, it has no `schema`, or to be precisely `schema` is synonymous to `database`. `database` is a kind of namespace containing common database objects like tables, indexes, foreign keys etc. A MySQL instance can hold many databases which are isolated to each other.

`User` in MySQL is called `user account`, which consists of 2 parts: user name and host name, because of this we can have finer control over privileges when same user name connect from different hosts, and typically we need to distinguish between localhost, hosts within the same private network, and public Internet addresses.

MySQL has a default administrative user `root`, for self-hosted MySQL instance, you need to provide root password during the installation process, either by config file or command line prompt. With `root` user, you can create databases and more administrative users, and typically you need to create several normal users with limited privileges for daily use and application use.

```sql
-- connect to mysql instance with root user
mysql -h 127.0.0.1 -u root -p

-- create a database with charset set to utf8
CREATE DATABASE db_name CHARACTER SET utf8 COLLATE utf8_general_ci;

-- create an account: finley@localhost
CREATE USER 'finley'@'localhost' IDENTIFIED BY 'some_pass';

-- grant all privileges on database db_name to finley@localhost
GRANT ALL PRIVILEGES ON db_name.* TO 'finley'@'localhost';

FLUSH PRIVILEGES;
```

As for Saas MySQL instance, generally venders won't give you the root password, and you need to create databases and normal users through Web UI.



## PostgreSQL

PostgreSQL always have better compliance with SQL standard comparing to MySQL, for instance, it has `schema` with a different meaning from `database`. A PostgreSQL database cluster contains multiple named `database`s, which are also isolated to each other. And a `database` contains more or more schemas, which in turn contains tables and other common database objects. `schema` of PostgreSQL means to organize database objects into logical groups, make them more manageable, and allows different users connect to the **same database** without interfering with each other.

You can CRUD and use schemas like databases or tables:
```sql
CREATE SCHEMA myschema;
CREATE TABLE myschema.mytable (
 ...
);
DROP SCHEMA myschema;
```

`Schema`s in PostgreSQL could be invisible if you do not pay enough attention, because PostgreSQL has 2 very important concepts: `public schema` and `schema search path`. If you create tables without specifying schema names, they go to the schema named `public` by default, and every database contains such a schema unless otherwise explicitly dropped. If you omit schema names when you refer a table, PostgreSQL determines which actual table is by following the `schema search path`, which is a list of a schema to look at, the first found is returned. 

The default `schema search path` is `"$user",public`, so the schema with the same name as the current user is searched first, then public schema. So if you want to kill `schema` in PostgreSQL and let all users connected to the database share everything, don't create any schemas and you have it by default. If you want to isolate each user from each other, create a schema for each user with the exact user name, and better off dropping the public schema.

Since PostgreSQL 8.1, `user` and `group` have been replaced by the merged concept `role`, the administrative role is `superuser`. For a SaaS PostgreSQL instance, superuser is generally not provided just like MySQL, but you can accomplish most of the operation work through Web UI, like create new user, configure whitelist for remote connection, With these two you should be able to obtain a database connection, then you can carry on creating databases and manipulating schemas via the command line PostgreSQL client `psql`

Things get a little more complicated for self-hosted PostgreSQL instances, and you get the chance to touch the details of Postgres' authenticate protocols. After PostgreSQL has been installed on a Linux machine, you need to run `# postgresql-setup initdb` to initiate Postgres, this includes a lot of stuff which all mean to make Postgres ready to use.

PostgreSQL supports a variety of authentication methods and options, and the configuration file for that is `pg_hba.conf`, for CentOS it resides in `/var/lib/pgsql/data`. You can configure authentication method for incoming connection attempts either from local UNIX sockets or remote SSL/non-SSL TCP sockets, and also for different databases and different roles. The authenticate method could be `trust` - absolute trust, no auth needed, `md5` and `password` - client send password across the connection either MD5-hashed or in plain text, and `peer` and `ident` which grab the underlying OS user name for authentication. Kerberos, LDAP, RADIUS and many other industry-level authentication solutions can also be integrated, you can read the official documentation [here](https://www.postgresql.org/docs/current/static/auth-methods.html).


```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            ident
#host    replication     postgres        ::1/128                 ident                                           
```

Freshly installed and configured PostgreSQL adds a user named `postgres` to the underlying OS, and this corresponds to the very first role in Postgres, which is also a `superuser`. From here you create more roles and assign privileges to them, and configure appropriate authentication methods in `pg_hba.conf`.

Many client apps including `psql` implicitly take current OS user name as the `role` for establishing Postgres connections, so `role`s could be invisible just like `schema`s in Postgres, be careful of that.


## Oracle

Oracle also has `schema`s under `database`s as logical containers for database objects, but they are strictly bond to `user`s, which means every time you create a user for Oracle, a `schema` is created automatically. Generally you go to your own schema after a database connection is established to Oracle, but you can alter session to another schema, or select data from other schemas if you have privileges granted.


My experience with Oracle is all with big enterprises, when we need an Oracle instance, we file a request, then DBA department assigns us a TNS connect string with everything behind it ready. Nonetheless I've ever tried to install Oracle Express Edition manually on a Linux machine, and it was quite miserable. It would be way much easier now as we have Docker and DSC tools at our hands, and I have an Ansible script [here](https://gist.github.com/lxdcn/5bd7114070ae3120f126).



<br /><br />
### Refers

- https://dev.mysql.com/doc/refman/5.7/en/glossary.html
- https://www.postgresql.org/docs/9.6/static/ddl-schemas.html
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7
- https://www.postgresql.org/docs/current/static/user-manag.html
- https://www.postgresql.org/docs/current/static/database-roles.html
- https://docs.oracle.com/database/121/CNCPT/intro.htm#CNCPT940


---
layout: default
title: SQL
license: not needed
---

## Introduction

Your aim in this lab is to become familar with the management and use
of simple databases using SQL. You will use the [MySQL](https://www.mysql.com/)
database management system (DBMS) for the practical work. MySQL is the
most widely used DBMS in the world. The lab assumes that you are working
in a Linux environment (Ubuntu 16.04) but most methods that you see here
will work in any environment (e.g. Mac OS/X, Windows, etc).

At the end of this lab, you should know how to:

* install MySQL
* create a database
* add tables to a database
* insert data into tables
* use SELECT...FROM...WHERE (SFW) to access data from a single table
* access data from multiple tables
* summarise data
* delete data
* create a new user of the MySQL server and grant permissions

## Installation of MySQL

1. Firstly, check to see if MySQL is already installed on your system:

     ```sh
     $ dpkg -l | grep mysql
     ```
   If you see output similar to the following

     ```
     ii  mysql-client-5.7        5.7.21-0ubuntu0.16.04.1  amd64 MySQL database client binaries
     ii  mysql-client-core-5.7   5.7.21-0ubuntu0.16.04.1  amd64        MySQL database core client binaries
     ii  mysql-common            5.7.21-0ubuntu0.16.04.1  all          MySQL database common files, e.g. /etc/mysql/my.cnf
     ii  mysql-server            5.7.21-0ubuntu0.16.04.1  all          MySQL database server (metapackage depending on the latest version)
     ii  mysql-server-5.7        5.7.21-0ubuntu0.16.04.1  amd64        MySQL database server binaries and system database setup
     ii  mysql-server-core-5.7   5.7.21-0ubuntu0.16.04.1  amd64        MySQL database server binaries
     ```
   then you already have the MySQL components installed and you can skip the next step.

1. If MySQL is not already installed, install it as follows:

     ```sh
     $ sudo apt-get install mysql-server
     ```
   During installation you will be prompted to enter a password for the root
   user. This is referring to the root user of the MySQL server, not the
   root user of your OS. Choose a password and make sure that you
   remember it. If you are working in a University lab, we suggest that you
   use the same password that you use to log in to your OS.

1. You can check that MySQL has been installed and the server has been started:

     ```sh
     $ systemctl status mysql.service
     ```
   You should see some output like this:

     ```
     mysql.service - MySQL Community Server
       Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
       Active: active (running) since Sat 2018-02-17 20:00:28 GMT; 14h ago
     Main PID: 31048 (mysqld)
        Tasks: 28
       Memory: 130.3M
          CPU: 24.616s
      CGroup: /system.slice/mysql.service
              └─31048 /usr/sbin/mysqld

     Feb 17 20:00:26 red systemd[1]: Starting MySQL Community Server...
     Feb 17 20:00:28 red systemd[1]: Started MySQL Community Server.
     ```

1. You can stop, start, or restart the MySQL server:

     ```sh
     $ sudo systemctl stop mysql.service
     $ sudo systemctl start mysql.service
     $ sudo systemctl restart mysql.service
     ```

## Connecting to and disconnecting from the server

1. We can connect to the server as the root user like this:

     ```sh
     $ mysql -u root -p
     Enter password: ***********
     ```
   Use the root password at the `Enter password:` prompt.
   When you have connected, you should see some output like this:

     ```
     Welcome to the MySQL monitor.  Commands end with ; or \g.
     Your MySQL connection id is 4
     Server version: 5.7.21-0ubuntu0.16.04.1 (Ubuntu)

     Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

     Oracle is a registered trademark of Oracle Corporation and/or its
     affiliates. Other names may be trademarks of their respective
     owners.

     Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

     mysql>
     ```
   The prompt `mysql>` shows that your subsequent commands will be sent to 
   the MySQL server.

1. You can disconnect from the MySQL server:

     ```sh
     mysql> QUIT;
     Bye
     $
     ```
   or

     ```sh
     mysql> \q 
     Bye
     $
     ```
   There's more information about connecting to and disconnecting from a MySQL
   server in the [reference
   manual](https://dev.mysql.com/doc/refman/5.7/en/connecting-disconnecting.html).
   We'll assume that you're connected to the MySQL server as the root user in
   the rest of these exercises. You would not normally work like this but it
   avoids the need to talk about users and permissions. We'll show you how to
   create other users and assign permissions to them at the end.

## Creating a new user and granting permissions

1. Work through [Etel Sverdlov's
   tutorial](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)
   on this topic.

1. As the root user, 
    * create a database called KF4005
    * create a MySQL user called `student` for the server on the `localhost`,
   with the same password for the `student` user as they would use to log in
   to the OS
    * ensure that the student user has all privileges in the KF4005 database but
   no privileges to do anything else






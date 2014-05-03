check_mysql_threads
===================

Nagios plugin for MySQL.

Checks the MySQL processlist for long running threads.

Threads which have been running for a long time could signal that something
is very wrong within MySQL, such as hung queries wasting CPU cycles.

This could be caused by application contention, for example,
when trying to acquire locks on the same rows.

However, long running threads could also be caused by DDL statements,
such as running ALTER on large tables.

This plugin can be helpful in monitoring the overall health
of an infrastructure, when combined with other standard checks.

For example, a webserver might not be responding to clients' requests.

In such situations, it's tempting to focus only on the webserver,
to try and figure out what is wrong.

Although in fact, it might be MySQL who is causing the actual problem,
because it can't keep up with SQL queries from the application.


Requirements
------------

This plugin should be compatible with any UNIX-like system,
as long as bourne shell is available and the following requirements are met;

1. The standard nagios-plugins are installed.

	This is required since this plugin sources utils.sh.

2. The MySQL command-line tool (client) is installed on
	the system where this plugin will be executed.

3. The MySQL server is running version 5.1.7 or later.

	This is required because older versions do not provide
	the `information_schema`.`PROCESSLIST` table.

4. The MySQL user configured for this plugin must be granted
	the PROCESS privilege.

	This is required so that the plugin checks *all* the threads running
	on the server, and not just the ones owned by the specific user.


Installation
------------

These installation instructions are targeted towards Ubuntu.

We also assume that the MySQL server we shall monitor is running locally.

Requirements.
```
apt-get install nagios-plugins mysql-client
```

Copy `check_mysql_threads` to `/usr/lib/nagios/plugins/check_mysql_threads`

Set ownership and permissions;
```
chown root:root /usr/lib/nagios/plugins/check_mysql_threads
chmod 0755      /usr/lib/nagios/plugins/check_mysql_threads
```

Login to MySQL server (as root) and create a separate user for this plugin;
```
CREATE USER 'nagios'@'localhost' IDENTIFIED BY 'secret';
GRANT PROCESS ON *.* TO 'nagios'@'localhost';
FLUSH PRIVILEGES;
```

Write a defaults file to `/etc/nagios/mysql.cnf`
```
[client]
host=localhost
port=3306
user=nagios
password=secret
```

We should finally be able to execute this check.

The following will exit with WARNING if threads have been running
for longer than 5 minutes, or CRITICAL for over 10 minutes.
```
~$ /usr/lib/nagios/plugins/check_mysql_threads --defaults-file /etc/nagios/mysql.cnf -w 300 -c 600
OK - no long running threads
~$ echo $?
0
```

Notes
-----

Threads executing the following commands will be ignored by the plugin,
as they can be very long lived;

1. Binlog Dump

	This is a thread on a master server for sending
	binary log contents to a slave server.

2. Connect

	A replication slave is connected to its master.

3. Sleep

	The thread is waiting for the client to send
	a new statement to it (persistent connection).


Testing
-------

Tested on Ubuntu 12.04.4 LTS running MySQL 5.5.34.

If you can confirm this plugin to work with other versions or configurations,
please contact me so I can update this section. Thank you!

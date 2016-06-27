# Overview

[MariaDB](https://mariadb.org) strives to be the logical choice for database
professionals looking for a robust, scalable, and reliable SQL server. To
accomplish this, the MariaDB Foundation works closely and cooperatively with
the larger community of users and developers in the true spirit of Free and
open source software, and release software in a manner that balances
predictability with reliability.

[MariaDB Enterprise](https://mariadb.com) from MariaDB Corporation, Inc. takes
MariaDB and enhances it with an optimized configuration, additional testing,
and available 24/7 professional support and consulting.

This charm deploys either MariaDB, using packages provided by the MariaDB
Foundation, including packages for IBM's POWER8 platform; or MariaDB
Enterprise, using packages in a repository provided by MariaDB Corporation,
Inc.

Note: Packages for IBM's POWER8 platform are only available from the MariaDB
Foundation repository.

As much as possible this charm uses the same charm structure as the MySQL charm
for the sake of compatability.

# Usage

## General Usage

### Deploying MariaDB
To deploy MariaDB from the MariaDB Foundation, simply deploy like so:

    juju deploy mariadb

MariaDB will be deployed.

### Deploying MariaDB Enterprise
To deploy a MariaDB Enterprise service, first login to the
[MariaDB Portal](https://mariadb.com/my_portal). On that page you will find
your MariaDB Enterprise download token. It is of the form `xxxx-xxxx`.

Next create a file called enterprise.yaml with the following contents,
replacing the placeholder token with your actual token:

    mariadb:
      enterprise-eula: true
      token: "xxxx-xxxx"

Lastly, deploy MariaDB Enterprise like so:

    juju deploy --config ./enterprise.yaml mariadb

MariaDB Enterprise will be deployed. You must agree to all terms contained in
`ENTERPRISE-LICENSE.md` in the charm directory to use MariaDB Enterprise.

### Installing a different series of MariaDB
Different series of MariaDB can be installed using the charm. The default
series is MariaDB 10.1, but the older MariaDB 5.5 and 10.0 are also available.
To install one of these older series, create a mariadb.yaml file with the
following contents (example is for MariaDB 5.5):

    mariadb:
      series: "5.5"

You could also add the series line to your enterprise.yaml file, if you are
using MariaDB Enterprise. Then when deploying MariaDB, reference the file like
so:

    juju deploy --config ./mariadb.yaml mariadb

Warning: Using the set command to downgrade MariaDB after initial deployment, for example, like so:

    juju set mariadb series="5.5"

...does not work. Using the set command to upgrade MariaDB; from 5.5 to 10.0,
or from 10.0 to 10.1; does work.


### After deploying
Once deployed, you can retrive the MariaDB root user password by logging in to
the machine via `juju ssh` and reading the `/var/lib/mysql/mysql.passwd` file.
To log in as the root MariaDB user at the MariaDB console you can, for example,
issue the following:

    juju ssh mariadb/0
    mysql -u root -p$(sudo cat /var/lib/mysql/mysql.passwd)

# Scale Out Usage

## Replication

MariaDB supports the ability to replicate databases to slave instances. This
allows you, for example, to load balance read queries across multiple slaves or
use a slave to perform backups, all whilst not impeding the master's
performance.

To deploy a slave:

    # deploy second service
    juju deploy --config ./enterprise.yaml mariadb mariadb-slave

    # add master to slave relation
    juju add-relation mariadb:master mariadb-slave:slave

Any changes to the master are reflected on the slave.

Any queries that modify the database(s) should be applied to the master only.
The slave should be treated strictly as read only.

You can add further slaves with:

    juju add-unit mariadb-slave

## Monitoring

This charm provides relations that support monitoring via either
[Nagios](https://jujucharms.com/precise/nagios) or
[Munin](https://jujucharms.com/precise/munin/). Refer to the appropriate charm
for usage.


# Configuration

You can tweak various options to optimize your MariaDB deployment:

* max-connections - Maximum connections allowed to server or '-1' for default.

* preferred-storage-engine - A comma separated list of storage engines to
  optimize for. First in the list is marked as default storage engine. 'InnoDB'
  or 'MyISAM' are acceptable values.

* tuning-level - Specify 'safest', 'fast' or 'unsafe' to choose required
  transaction safety. This option determines the flush value for innodb commit
  and binary logs. Specify 'safest' for full ACID compliance. 'fast' relaxes the
  compliance for performance and 'unsafe' will remove most restrictions.

* dataset-size - Memory allocation for all caches (InnoDB buffer pool, MyISAM
  key, query). Suffix value with 'K', 'M', 'G' or 'T' to indicate unit of
  kilobyte, megabyte, gigabyte or terabyte respectively. Suffix value with '%'
  to use percentage of machine's total memory.

* query-cache-type - Specify 'ON', 'DEMAND' or 'OFF' to turn query cache on,
  selectively (dependent on queries) or off.

* query-cache-size - Size of query cache (no. of bytes) or '-1' to use 20%
  of memory allocation.

Each of these can be applied by running:

    juju set <service> <option>=<value>

For example:

    juju set mariadb preferred-storage-engine=InnoDB
    juju set mariadb dataset-size=50%
    juju set mariadb query-cache-type=ON
    juju set mariadb query-cache-size=-1


# MariaDB Contact Information

- [MariaDB Homepage](http://mariadb.org)
- [MariaDB Enterprise Homepage](http://mariadb.com)
- [MariaDB Bug Tracker](https://mariadb.org/jira)
- [MariaDB mailing lists and other user resources](https://mariadb.com/kb/en/resources/)


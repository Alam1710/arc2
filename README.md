# ARC2

[![Build](https://travis-ci.org/semsol/arc2.svg?branch=master)](https://travis-ci.org/semsol/arc2)
[![Coverage Status](https://coveralls.io/repos/github/semsol/arc2/badge.svg?branch=master)](https://coveralls.io/github/semsol/arc2?branch=master)
[![Latest Stable Version](https://poser.pugx.org/semsol/arc2/v/stable.svg)](https://packagist.org/packages/semsol/arc2)
[![Total Downloads](https://poser.pugx.org/semsol/arc2/downloads.svg)](https://packagist.org/packages/semsol/arc2)
[![Latest Unstable Version](https://poser.pugx.org/semsol/arc2/v/unstable.svg)](https://packagist.org/packages/semsol/arc2)
[![License](https://poser.pugx.org/semsol/arc2/license.svg)](https://packagist.org/packages/semsol/arc2)

ARC2 is a PHP 5.6+ library for working with RDF. It also provides a MySQL-based triplestore with SPARQL support.

## Installation

Package available on [Composer](https://packagist.org/packages/semsol/arc2).

If you're using Composer to manage dependencies, you can use

```bash
composer require semsol/arc2:2.4.*
```

### Branches

`2.3.1` was latest stable version for a long time. But recent developments lead to version `2.4`, which is the next minor version of the 2.x-branch, containing stabilizations, wider test coverage and a couple of new features (e.g. PDO adapter for RDF store). Primary focus was to keep backward compatibility, so that an upgrade doesn't mess up your application. Its not clear how long the 2.x-branch will be maintained, so please consider upgrading to 3.x.

![](doc/branches.png)

Version `3.x` introduces new features and develops the backend further. Unfortunately, backward compatibility can not be maintained. One of the major changes is the transition from MyISAM to InnoDB table engine (RDF store).

## Requirements

#### PHP

|          5.6          |        7.0         |        7.1         |        7.2         |
|:---------------------:|:------------------:|:------------------:|:------------------:|
| :heavy_check_mark:(1) | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |

(1) It is compatible with PHP 5.3+ but old versions are no longer tested.

#### Database systems

|           |        5.5         |        5.6         |        5.7         |       8.0       |
|:---------:|:------------------:|:------------------:|:------------------:|:---------------:|
| **MySQL** | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :collision: (1) |

|             |        10.0        |        10.1        |        10.2        |        10.3        |
|:-----------:|:------------------:|:------------------:|:------------------:|:------------------:|
| **MariaDB** | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |

(1) As long as ARC2 uses mysqli, a connection to MySQL Server 8.0 is not possible. For more information, please look [here](https://github.com/semsol/arc2/commit/0ad48d61753b15ae02ff19f615b14aa52b6557f1). But its planned to switch to PDO ([issue](https://github.com/semsol/arc2/issues/109))


## RDF triple store

### SPARQL support

Please have a look into [SPARQL-support.md](doc/SPARQL-support.md) to see which SPARQL 1.0/1.1 features are currently supported.

### Transaction support

With the PDO adapter, ARC2 provides extended transaction support. This means, that one can also use nested transactions. This is very helpful, if the ARC2 database connection is shared by different applications in one session. Our adapter allows the creation of transactions as you need it without the need to maintain them yourself.

**Short example:**

```php
$store = ARC2::getStore(array(
    'db_name' => 'testdb',
    'db_user' => 'root',
    'db_pwd'  => '',
    'db_host' => '127.0.0.1',
    // use PDO, because mysqli adapter does not support transactions!
    'db_adapter' => 'pdo',
    'db_pdo_protocol' => 'mysql'
));

try {
    $store->beginTransaction();

    // do something

    $store->commit();
} catch (\Exception $e) {
    $store->rollback();
}
```

Transaction support is implemented using PDOs own implementations as well as use queries to setup the server, e.g. by sending `SET autocommit=0;` ([source](https://dev.mysql.com/doc/refman/5.5/en/innodb-autocommit-commit-rollback.html)). Also important: transactions can be used without any setup, just call `beginTransaction`, `commit`, `inTransaction` or `rollback` as usual.

### Known problems/restrictions with database systems

In this section you find known problems with MariaDB or MySQL, regarding certain features. E.g. MySQL 5.5 doesn't allow FULLTEXT indexes in InnoDB. We try to encapsulate any differences in the DB adapters, so that you don't have to care about them. In case you run into problems, this section might be of help.

#### MySQL 5.5

MySQL 5.5 does not support FULLTEXT for InnoDB ([Source](https://dev.mysql.com/doc/refman/5.5/en/fulltext-restrictions.html)):

> Full-text searches are supported for MyISAM tables only. (In MySQL 5.6 and up, they can also be used with InnoDB tables.)

#### MySQL 8.0 and mysqli

Using mysqli with MySQL 8.0 as backend throws the following exception:

> mysqli_connect(): The server requested authentication method unknown to the client [caching_sha2_password]

Based on this [source](https://mysqlserverteam.com/upgrading-to-mysql-8-0-default-authentication-plugin-considerations/), one has to change the my.cnf, adding the following entry:

> [mysqld]
> default-authentication-plugin=mysql_native_password

## Internal information for developers

Please have a look [here](doc/developer.md) to find information about maintaining and extending ARC2.

### Docker setup

For ARC2 developers we recommend this following [Docker setup](https://github.com/k00ni/PHP-Apache-MySQL-Docker). It provides a pre-configured set of software (for PHP, DBS etc.) and allows quick switches between different software versions.

For more information have a look [here](https://github.com/k00ni/PHP-Apache-MySQL-Docker).

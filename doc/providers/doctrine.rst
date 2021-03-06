DoctrineServiceProvider
=======================

The *DoctrineServiceProvider* provides integration with the `Doctrine DBAL
<http://www.doctrine-project.org/projects/dbal>`_ for easy database acccess.

.. note::

    There is only a Doctrine DBAL. An ORM service is **not** supplied.

Parameters
----------

* **db.options**: Array of Doctrine DBAL options.

  These options are available:

  * **driver**: The database driver to use, defaults to ``pdo_mysql``.
    Can be any of: ``pdo_mysql``, ``pdo_sqlite``, ``pdo_pgsql``,
    ``pdo_oci``, ``oci8``, ``ibm_db2``, ``pdo_ibm``, ``pdo_sqlsrv``.

  * **dbname**: The name of the database to connect to.

  * **host**: The host of the database to connect to. Defaults to
    localhost.

  * **user**: The user of the database to connect to. Defaults to
    root.

  * **password**: The password of the database to connect to.

  * **path**: Only relevant for ``pdo_sqlite``, specifies the path to
    the SQLite database.

  These and additional options are described in detail in the `Doctrine DBAL
  configuration documentation <http://www.doctrine-project.org/docs/dbal/2.0/en/reference/configuration.html>`_.

* **db.dbal.class_path** (optional): Path to where the
  Doctrine DBAL is located.

* **db.common.class_path** (optional): Path to where
  Doctrine Common is located.

Services
--------

* **db**: The database connection, instance of
  ``Doctrine\DBAL\Connection``.

* **db.config**: Configuration object for Doctrine. Defaults to
  an empty ``Doctrine\DBAL\Configuration``.

* **db.event_manager**: Event Manager for Doctrine.

Registering
-----------

Make sure you place a copy of *Doctrine DBAL* in ``vendor/doctrine-dbal``
and *Doctrine Common* in ``vendor/doctrine-common``::

    $app->register(new Silex\Provider\DoctrineServiceProvider(), array(
        'db.options'            => array(
            'driver'    => 'pdo_sqlite',
            'path'      => __DIR__.'/app.db',
        ),
        'db.dbal.class_path'    => __DIR__.'/vendor/doctrine-dbal/lib',
        'db.common.class_path'  => __DIR__.'/vendor/doctrine-common/lib',
    ));

Usage
-----

The Doctrine provider provides a ``db`` service. Here is a usage
example::

    $app->get('/blog/show/{id}', function ($id) use ($app) {
        $sql = "SELECT * FROM posts WHERE id = ?";
        $post = $app['db']->fetchAssoc($sql, array((int) $id));

        return  "<h1>{$post['title']}</h1>".
                "<p>{$post['body']}</p>";
    });

Using multiple databases
------------------------

The Doctrine provider can allow access to multiple databases. In order to
configure the data sources, replace the **db.options** with **dbs.options**.
**dbs.options** is an array of configurations where keys are connection names
and values are options::

    $app->register(new Silex\Provider\DoctrineServiceProvider(), array(
        'dbs.options' => array (
            'mysql_read' => array(
                'driver'    => 'pdo_mysql',
                'host'      => 'mysql_read.someplace.tld'
                'dbname'    => 'my_database',
                'user'      => 'my_username',
                'password'  => 'my_password',
            ),
            'mysql_write' => array(
                'driver'    => 'pdo_mysql',
                'host'      => 'mysql_write.someplace.tld'
                'dbname'    => 'my_database',
                'user'      => 'my_username',
                'password'  => 'my_password',
            ),
        ),
        'db.dbal.class_path'    => __DIR__.'/vendor/doctrine-dbal/lib',
        'db.common.class_path'  => __DIR__.'/vendor/doctrine-common/lib',
    ));

The first registered connection is the default and can simply be accessed as
you would if there was only one connection. Given the above configuration,
these two lines are equivalent::

    $app['db']->fetchAssoc('SELECT * FROM table');

    $app['dbs']['mysql_read']->fetchAssoc('SELECT * FROM table');

Using multiple connections::

    $app->get('/blog/show/{id}', function ($id) use ($app) {
        $sql = "SELECT * FROM posts WHERE id = ?";
        $post = $app['dbs']['mysql_read']->fetchAssoc($mysqlQuery, array((int) $id));

        $mysqlUpdate = "UPDATE posts SET value = ? WHERE id = ?";
        $app['dbs']['mysql_write']->execute($mysqlUpdate, array('newValue', (int) $id));

        return  "<h1>{$post['title']}</h1>".
                "<p>{$post['body']}</p>";
    });

For more information, consult the `Doctrine DBAL documentation
<http://www.doctrine-project.org/docs/dbal/2.0/en/>`_.

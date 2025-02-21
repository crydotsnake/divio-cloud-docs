.. _interact-database:

How to interact with your application's database
================================================

The database for your Divio application runs:

* in a Docker container for your **local** projects: :ref:`interact-local-db`
* on a dedicated cluster for your **cloud-deployed** sites: :ref:`interact-cloud-db`

In either case, you will mostly only need to interact with the database using the tools provided by
your project's runtime stack (e.g. Django). However, if you need to interact with it directly, the
option exists.

..  admonition:: The database service name in ``docker-compose.yml``

    The Divio CLI expects that the database service will be named ``database_default`` (or ``db``) in your
    ``docker-compose.yml`` file. If not, it certain commands (such as ``divio app push/pull db``) will fail.


.. _interact-local-db:

Interact with the local database
--------------------------------

Generally, the most convenient way to interact with the application's database is to do it locally (with a local
copy of your cloud data if necessary).


From the project's local Django web container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using ``dbshell``
^^^^^^^^^^^^^^^^^

Run::

    docker-compose run --rm web python ./manage.py dbshell


Connecting to a Postgres database manually
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also make the connection manually from within the ``web`` container, for example::

    docker-compose run --rm web psql -h database_default -U postgres db

The ``-h`` value (for host) needs to match the name of the database service in the ``docker-compose.yml`` file, which
might be different (for example, ``database_default``).

As well as ``psql`` you can run commands such as ``pg_dump`` and ``pg_restore``. This is useful
for a number of :ref:`common operations <common-db-operations>`, below.

Your project may not have the ``psql`` client installed already, in which case you will need to install it first. See
:ref:`install-system-packages`.


Using ``docker exec``
.....................

Another way of interacting with the database is via the database container itself, using ``docker
exec``. This requires that the database container already be up and running.

For example, if your database container is called ``example_database_default_1``::

    docker exec -i example_database_default_1 psql -U postgres


From your host environment
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have a preferred database management tool that runs on your own computer, you can also
connect to the database from outside the application.


.. _expose-database-ports:

Expose the database's port to the host
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to the connect to the database from a tool running directly on your
own machine, you will need to expose its port (5432 by default for Postgres).

Add a ports section to the database service in ``docker-compose.yml`` and map the
port to your host. For Postgres, for example:

..  code-block:: yaml
    :emphasize-lines: 3,4

    database_default:
        image: postgres:13.5-alpine
        ports:
            - 5432:5432

This means that external traffic reaching the container on port 5432 will be
routed to port 5432 internally.

The ports are ``<host port>:<container port>`` - you can choose another host
port if you are already using that port on your host.

Now restart the database container with: ``docker-compose up -d database_default``


Connect to a Postgres database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You will need to use the following details:

* port: ``5432``
* username: ``postgres``
* password: not required
* database: ``db``

Access the database using your Postgres tool of choice. Note that you must
specify the host address, ``127.0.0.1``.

For example, if you're using the ``psql`` command line tool, you can connect to the project
database with::

    psql -h 127.0.0.1 -U postgres db


.. _interact-cloud-db:

Interact with the Cloud database
--------------------------------

Use the ``divio app pull db`` and ``divio app push db`` commands to copy a database between a cloud environment
and your own local environment.

Note that the ``pull`` operation downloads a binary database dump (in a tarred archive), whereas ``push`` creates and
uploads a SQL database dump.

See the :ref:`divio CLI command reference <divio-cli-command-ref>` for more on using these commands.


From the project's Cloud application container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Log into your Cloud project's container (Test or Live) over SSH.


Using ``dbshell`` in a Django project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run::

    ./manage.py dbshell

This will drop you into a command-line client, connected to your database.


Connecting to a database manually
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also make the connection manually. Run ``env`` to list your environment variables. Amongst
them you'll find ``DATABASE_URL``, which will be in the form::

    schema://<user name>:<password>@<address>:<port>/<name>

You can use these credentials in the appropriate client, e.g. ``psql``.


From your own computer
~~~~~~~~~~~~~~~~~~~~~~

Access to cloud databases other than from the associated application containers is not possible -
it is restricted, for security reasons, to containers running on our own infrastructure.


.. _change-database-version:

Change the local database engine version
----------------------------------------

Sometimes, you will need to change the database engine, or its version number, that your local project uses
- for example if the cloud database is updated or changed. If the two database engines are not the
same, you may run into problems.

The local database engine is specified by the ``image`` option in the database service (usually called ``database_default`` in
your project's ``docker-compose.yml`` file, for example:

..  code-block:: yaml
    :emphasize-lines: 2

    database_default:
        image: postgres:13.5-alpine

Should you need to change this, that line should be updated - for example if the Cloud database is
now running Postgres 14:

..  code-block:: yaml
    :emphasize-lines: 2

    database_default:
        image: postgres:14-alpine

Docker will use the new version the next time the local project is launched.

If you are not sure what image to use for the local database, Divio support will be able to advise
you.

..  important::

    In the Divio architecture, the ``docker-compose.yml`` file is **not**
    used for Cloud deployments, but **only** for the local server. The changes you
    make here will not affect the Cloud database.


.. _manage_postgres_extensions:

Manage Postgres extensions
--------------------------

Although you cannot create extensions yourself on our shared database clusters, we can often enable extensions for you
on request. The most commonly-requested of these is `PostGIS <https://postgis.net>`_. Please contact Divio support
for this.

You will run into errors if you perform an operation that requires or tries to create a missing extension, for example:

..  code-block:: text

    psycopg2.errors.InsufficientPrivilege: permission denied to create extension "unaccent"

from a database migration or

..  code-block:: text

    ---> Processing error!

from a ``divio push db`` command, when the local database uses an extension not available on the cloud.

Run the Postgres ``\dx`` command :ref:`in a local database shell <interact-local-db>` or in a cloud shell to list
extensions that you're using.


.. _common-db-operations:

Usage examples for common basic operations
------------------------------------------

It's beyond the scope of this article to give general guidance on using the database, but these
examples will help give you an idea of some typical operations that you might undertake while using
Divio.

All the examples assume that you are interacting with the local database, running in its  ``db``
container, and will use Postgres.

In each case, we launch the command from within the ``web`` container with ``docker-compose run
--rm web`` and we specify:

* host name: ``-h database_default``
* user name: ``-U postgres``


.. _dump-db:

Dump the database
~~~~~~~~~~~~~~~~~

From the ``web`` service, dump the database ``db`` to a file named ``database.dump``:

..  code-block:: bash

    docker-compose run --rm web pg_dump -h database_default -U postgres db > database.dump


.. _drop-db:

Drop the database
~~~~~~~~~~~~~~~~~

Drop (delete) the database named ``db``:

..  code-block:: bash

    docker-compose run --rm web dropdb -h database_default -U postgres db


.. _create-db:

Create the database
~~~~~~~~~~~~~~~~~~~~~

Create a database named ``db``:

..  code-block:: bash

    docker-compose run --rm web createdb -h database_default -U postgres db


.. _apply-hstore-db:

Apply the ``hstore`` extension
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Apply the ``hstore`` extension (required on a newly-created local database) to the database named
``db``:

..  code-block:: bash

    docker-compose run --rm web psql -h database_default -U postgres db -c "CREATE EXTENSION hstore"


.. _restore-db:

Restore the database
~~~~~~~~~~~~~~~~~~~~

Restore a database named ``db`` from a file named ``database.dump``:

..  code-block:: bash

    docker-compose run --rm web pg_restore -h database_default -U postgres -d db database.dump --no-owner


.. _reset-database:

Reset the database
~~~~~~~~~~~~~~~~~~

To reset the database (with empty tables, but the schema in place) you would run the commands above
to :ref:`drop <drop-db>` and :ref:`create <create-db>` the database, :ref:`create the the hstore
extension <apply-hstore-db>`, followed by a migration::

    docker-compose run --rm web python manage.py migrate


Restore from a downloaded Cloud backup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Untar the downloaded ``backup.tar`` file. It contains a ``database.dump`` file. Copy the file to
your local project directory, then run the commands above to :ref:`drop <drop-db>` and :ref:`create
<create-db>` the database, :ref:`create the the hstore extension <apply-hstore-db>`, and then
:ref:`restore from a file <restore-db>`.

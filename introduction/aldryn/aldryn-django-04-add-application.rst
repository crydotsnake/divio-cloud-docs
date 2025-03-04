:sequential_nav: both

.. _tutorial-add-applications:

Make changes and deploy them
===================================

Next, we're going to install a new package, `Django Axes <https://github.com/jazzband/django-axes>`_, into the project
(Django Axes keeps track of log-in attempts). Then we'll test it and deploy it to the cloud.


.. _tutorial-add-requirements:

Install a package
-----------------

To be used in a containerised system, packages must be built into the image, otherwise the next time a container is
launched, the package will not be there. The image is built by the ``Dockerfile``, and in our ``Dockerfile`` for Django
projects, this includes an instruction to process the project's ``requirements.in`` file with Pip. This is where the
package needs to be added. Open ``requirements.in`` and at the end of it add a new line:

..  code-block:: bash

    django-axes==3.0.3

It's important to pin dependencies to a particular version this way; it helps ensure that we don't run into unwanted
surprises if the package is updated, and the new version introduces an incompatibility.

Now you can build the project again by running::

    docker-compose build


Configure the Django settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Django Axes requires that it be listed in the project's ``INSTALLED_APPS``, in ``settings.py``. Add ``axes`` to the
list:

..  code-block:: python
    :emphasize-lines: 4

    # all Django settings can be altered here

    INSTALLED_APPS.extend([
        "axes",
    ])


..  include:: /introduction/includes/04-add-application-django-01.rst


Run migrations
--------------

Django Axes introduces new database tables, so we need to run migrations:

..  code-block:: bash

    docker-compose run web python manage.py migrate

(As you have probably noticed, we can run all the usual Django management commands, but because we need to run them
inside the containerised environment, we precede each one with ``docker-compose run web``.)


Check the project
--------------------

If you launch the project again with ``docker-compose up`` you'll find Django Axes in the admin:

.. image:: /images/axes.png
   :alt: 'Django Axes in the admin'
   :width: 663

Test it by attempting to log in to the Django admin with an incorrect password.


Deploy to the cloud
-------------------

If you are satisfied with your work, you can deploy it to the cloud.

We made changes to two files (``requirements.in``, ``settings.py``). So:

..  code-block:: bash

    git add .
    git commit -m "Added Django Axes"
    git push

On the project Dashboard, you will see that your new commit is listed as *1 Undeployed commit*. You can deploy this
using the Control Panel, or by running:

..  code-block:: bash

    divio app deploy

When it has finished deploying, you should check the Test server to see that all is as expected. Once you're satisfied
that it works correctly, you can deploy the Live server too:

..  code-block:: bash

    divio app deploy live


..  include:: /introduction/includes/04-add-application-push-01.rst


--------------

:ref:`The next section <tutorial-application-configuration>` looks at some more complex configuration and application
integration.

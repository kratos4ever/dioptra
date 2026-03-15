.. This Software (Dioptra) is being made available as a public service by the
.. National Institute of Standards and Technology (NIST), an Agency of the United
.. States Department of Commerce. This software was developed in part by employees of
.. NIST and in part by NIST contractors. Copyright in portions of this software that
.. were developed by NIST contractors has been licensed or assigned to NIST. Pursuant
.. to Title 17 United States Code Section 105, works of NIST employees are not
.. subject to copyright protection in the United States. However, NIST may hold
.. international copyright in software created by its employees and domestic
.. copyright (or licensing rights) in portions of software that were assigned or
.. licensed to NIST. To the extent that NIST holds copyright in this software, it is
.. being made available under the Creative Commons Attribution 4.0 International
.. license (CC BY 4.0). The disclaimers of the CC BY 4.0 license apply to all parts
.. of the software developed or licensed by NIST.
..
.. ACCESS THE FULL CC BY 4.0 LICENSE HERE:
.. https://creativecommons.org/licenses/by/4.0/legalcode

.. _how-to-uninstall-dioptra:

Uninstall Dioptra
=================

This guide shows you how to completely remove a Dioptra deployment from your system, including containers, volumes, networks, and deployment configurations.

Prerequisites
-------------

* Terminal access to your Dioptra deployment directory
* Docker and Docker Compose installed
* Appropriate permissions to remove files and Docker resources

.. warning::

   Uninstalling Dioptra will permanently delete all experiment data, artifacts, databases, and configuration files. Make sure to back up any important data before proceeding.

Overview
--------

The uninstall process involves three main steps:

1. **Stop and Remove Containers** - Shut down all running Dioptra services
2. **Remove Persistent Data** - Delete Docker volumes containing databases and artifacts
3. **Clean Up Deployment Directory** - Remove the deployment configuration files

Choose the appropriate option based on what you need to remove:

**Option A: Complete Removal** - Use this when:

* You want to completely remove Dioptra and all associated data
* You're decommissioning a deployment permanently
* You need to free up all disk space used by Dioptra

**Option B: Preserve Data** - Use this when:

* You want to remove the deployment but keep experiment data
* You plan to reinstall Dioptra later with the same data
* You're troubleshooting and may need to restore the deployment

Option A: Complete Removal
---------------------------

Use this workflow when you want to completely remove Dioptra and all its data.

.. rst-class:: header-on-a-card header-steps

Step A1: Navigate to Your Deployment Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Navigate to your Dioptra deployment directory:

.. code:: sh

   cd /path/to/your/dioptra-deployment

Replace ``/path/to/your/dioptra-deployment`` with the actual path to your deployment.

.. rst-class:: header-on-a-card header-steps

Step A2: Stop and Disable the systemd Service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Skip this step if you did not install Dioptra as a systemd service.

If you installed the ``dioptra.service`` unit as described in
:ref:`reference-deployment-commands`, stop it and remove it before deleting the
deployment. Otherwise systemd will continue trying to start a deployment that no
longer exists.

.. code:: sh

   sudo systemctl stop dioptra
   sudo systemctl disable dioptra
   sudo rm /etc/systemd/system/dioptra.service
   sudo systemctl daemon-reload

.. rst-class:: header-on-a-card header-steps

Step A3: Stop and Remove All Services and Data Volumes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Stop all running containers and remove them along with their networks and volumes:

.. code:: sh

   docker compose down -v

The ``-v`` flag removes all volumes defined in the docker-compose configuration, including:

* PostgreSQL database
* Minio object storage
* MLflow tracking data
* Redis cache

.. warning::

   This step permanently deletes all experiment data, artifacts, model tracking information, and user accounts. This action cannot be undone.

.. rst-class:: header-on-a-card header-steps

Step A4: Remove the Deployment Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remove the deployment directory and all configuration files:

.. code:: sh

   cd ..
   rm -rf dioptra-deployment

Replace ``dioptra-deployment`` with your actual deployment directory name.

This removes:

* Docker Compose configuration files
* Environment variable files
* SSL/TLS certificates (if configured)
* Override configurations
* Initialization scripts

.. rst-class:: header-on-a-card header-steps

Step A5: (Optional) Remove Container Images
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you also want to remove the Dioptra container images to free up disk space:

.. code:: sh

   docker images | grep dioptra | awk '{print $3}' | xargs docker rmi

Alternatively, remove specific Dioptra images:

.. code:: sh

   docker rmi ghcr.io/usnistgov/dioptra/nginx:dev
   docker rmi ghcr.io/usnistgov/dioptra/mlflow-tracking:dev
   docker rmi ghcr.io/usnistgov/dioptra/restapi:dev
   docker rmi ghcr.io/usnistgov/dioptra/pytorch-cpu:dev
   docker rmi ghcr.io/usnistgov/dioptra/tensorflow2-cpu:dev

.. note::

   These examples use the ``dev`` tag, which is the default ``container_tag`` for a
   new deployment. Replace it with the tag your deployment actually uses, and add the
   GPU worker images (``tensorflow2-gpu``, ``pytorch-gpu``) if you deployed them. You
   can list all Dioptra images with ``docker images | grep dioptra``.

Option B: Preserve Data
-----------------------

Use this workflow when you want to stop and remove the deployment but preserve experiment data for later use.

.. rst-class:: header-on-a-card header-steps

Step B1: Stop and Remove Services (Preserve Volumes)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Navigate to your deployment directory and stop the services without removing volumes:

.. code:: sh

   cd /path/to/your/dioptra-deployment
   docker compose down

This stops all containers but preserves the Docker volumes containing your data.

If you installed Dioptra as a systemd service, stop it first so that it does not
restart the deployment:

.. code:: sh

   sudo systemctl stop dioptra
   sudo systemctl disable dioptra

.. rst-class:: header-on-a-card header-steps

Step B2: Record Volume Names for Later Use
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Record the volume names before you remove anything else, so you can identify your
preserved data when you create a new deployment:

.. code:: sh

   docker volume ls | grep dioptra

Save the output for reference.

.. rst-class:: header-on-a-card header-steps

Step B3: (Optional) Remove the Deployment Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to remove the configuration files but keep the data volumes:

.. code:: sh

   cd ..
   rm -rf dioptra-deployment

.. note::

   The Docker volumes will persist on your system even after removing the deployment
   directory. You can list them with ``docker volume ls | grep dioptra`` and manually
   remove them later if needed.

Verifying Removal
-----------------

After completing the uninstall, verify that Dioptra resources have been removed:

.. note::

   Docker names the containers, volumes, and networks after your deployment, so the
   examples below assume the default deployment name of ``Dioptra deployment``. If you
   chose a different name, substitute it in place of ``dioptra`` in the commands below.

**Check for running containers:**

.. code:: sh

   docker ps -a | grep dioptra

**Check for remaining volumes:**

.. code:: sh

   docker volume ls | grep dioptra

**Check for remaining networks:**

.. code:: sh

   docker network ls | grep dioptra

If any resources remain, you can manually remove them using:

.. code:: sh

   docker rm <container-name>
   docker volume rm <volume-name>
   docker network rm <network-name>

.. rst-class:: header-on-a-card header-seealso

See Also
--------

* :ref:`how-to-prepare-deployment` - Setting up a new Dioptra deployment
* :ref:`reference-deployment-commands` - Common deployment management commands
* :ref:`how-to-update-deployment` - Updating an existing deployment

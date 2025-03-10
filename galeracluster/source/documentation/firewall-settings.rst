.. meta::
   :title: Galera Cluster - Firewall Settings
   :description:
   :language: en-US
   :keywords: galera cluster, firewall, ports
   :copyright: Codership Oy, 2014 - 2025. All Rights Reserved.

.. container:: left-margin

   .. container:: left-margin-top

      :doc:`The Library <../index>`

   .. container:: left-margin-content

      .. cssclass:: here

         - :doc:`Documentation <./index>`

      - :doc:`Knowledge Base <../kb/index>`
      - :doc:`Training <../training/index>`

      .. cssclass:: sub-links

         - :doc:`Training Courses <../training/courses/index>`
         - :doc:`Tutorial Articles <../training/tutorials/index>`
         - :doc:`Training Videos <../training/videos/index>`

      - :doc:`FAQ <../faq>`
      - :ref:`search`

.. container:: top-links

   - `Home <https://galeracluster.com>`_

   .. cssclass:: here

      - :doc:`Docs <./index>`

   - :doc:`KB <../kb/index>`

   .. cssclass:: nav-wider

      - :doc:`Training <../training/index>`

   - :doc:`FAQ <../faq>`


.. cssclass:: library-document
.. _`firewall-settings`:

=====================
 Firewall Settings
=====================

Galera Cluster requires a number of ports to maintain network connectivity between the nodes. Depending on your deployment, you may not require all of these ports, but a cluster might require all of them on each node. Below is a list of these ports and their purpose:

- ``3306`` is the default port for MySQL client connections and :term:`State Snapshot Transfer` using ``mysqldump`` for backups.

- ``4567`` is reserved for Galera :term:`Cluster Replication` traffic. Multicast replication uses both TCP and UDP transport on this port.

- ``4568`` is the port for :term:`Incremental State Transfer`.

- ``4444`` is used for all other :term:`State Snapshot Transfer`.

How these ports are enabled for Galera Cluster can vary depending upon your operating system distribution and what you use to configure the firewall.

.. toctree::
   :maxdepth: 2

   ip-tables
   firewalld
   firewall-pf


.. only:: html

          .. image:: ../images/support.jpg
             :target: https://galeracluster.com/support/#galera-cluster-support-subscription
             :width: 740

   .. only:: latex

          .. image:: ../images/support.jpg
		  :target: https://galeracluster.com/support/#galera-cluster-support-subscription

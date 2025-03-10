.. meta::
   :title: Scriptable State Snapshot Transfers
   :description:
   :language: en-US
   :keywords: galera cluster, scriptable sst, state snapshot transfers
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

      Related Documents

      - :ref:`wsrep_sst_receive_address <wsrep_sst_receive_address>`
      - :ref:`wsrep_sst_auth <wsrep_sst_auth>`
      - :ref:`wsrep_sst_method <wsrep_sst_method>`

.. container:: top-links

   - `Home <https://galeracluster.com>`_

   .. cssclass:: here

      - :doc:`Docs <./index>`

   - :doc:`KB <../kb/index>`

   .. cssclass:: nav-wider

      - :doc:`Training <../training/index>`

   - :doc:`FAQ <../faq>`


.. cssclass:: library-document
.. _`scriptable-sst`:

=====================================
Scriptable State Snapshot Transfers
=====================================

When a node sends and receives a :term:`State Snapshot Transfer`, it manage it through processes that run external to the database server. If you need more from these processes than the default behavior provides, Galera Cluster provides an interface for custom shell scripts to manage state snapshot transfers on the node.


.. _`writing-custom-sst`:
.. rst-class:: section-heading
.. rubric:: Using the Common SST Script

Galera Cluster includes a common script for managing a :term:`State Snapshot Transfer`, which you can use as a starting point in building your own custom script. The filename is ``wsrep_sst_common``. For Linux users, the package manager typically installs it for you in ``/usr/bin``.

The common SST script provides ready functions for parsing argument lists, logging errors, and so on. There are no constraints on the order or number of parameters it takes. You can add new parameters and ignore any of the existing ones as you prefer.

It assumes that the storage engine initialization on the receiving node takes place only after the state transfer is complete. Meaning that it copies the contents of the source data directory to the destination data directory (with possible variations).


.. _`sst-script-parameters`:
.. rst-class:: section-heading
.. rubric:: State Transfer Script Parameters

When Galera Cluster starts an external process for state snapshot transfers, it passes a number of parameters to the script, which you can use in configuring your own state transfer script.


.. _`general-sst-script-parameters`:
.. rst-class:: sub-heading
.. rubric:: General Parameters

These parameters are passed to all state transfer scripts, regardless of method or whether the node is sending or receiving:

.. rst-class:: verbose-list

   ``--role`` The script is given a string, either ``donor`` or ``joiner``, to indicate whether the node is using it to send or receive a state snapshot transfer.

   ``--address`` The script is given the IP address of the :term:`Joiner Node`.

   When the script is run by the joiner, the node uses the value of either the :ref:`wsrep_sst_receive_address <wsrep_sst_receive_address>` parameter or a sensible default formatted as ``<ip_address>:<port>``. When the script is run by the donor, the node uses the value from the state transfer request.

   ``--auth`` The script is given the node authentication information.

   When the script is run by the joiner, the node uses the value given to the :ref:`wsrep_sst_auth <wsrep_sst_auth>` parameter. When the script is run by the donor, it uses the value given by the state transfer request.

   ``--datadir`` The script is given the path to the data directory. The value is drawn from the ``mysql_real_data_home`` parameter.

   ``--defaults-file`` The script is given the path to the ``my.cnf`` configuration file.

The values the node passes to these parameters varies depending on whether the node calls the script to send or receive a state snapshot transfer. For more information, see :ref:`Calling Conventions <calling-conventions>` below.


.. _`donor-sst-script-parameters`:
.. rst-class:: sub-heading
.. rubric:: Donor-specific Parameters

These parameters are passed only to state transfer scripts initiated by a node serving as the :term:`Donor Node`, regardless of the method being used:

.. rst-class:: verbose-list

   ``--gtid`` The node gives the :term:`Global Transaction ID`, which it forms from the state UUID and the sequence number, or seqno, of the last committed transaction.

   ``--socket`` The node gives the local server socket for communications, if required.

   ``--bypass`` The node specifies whether the script should skip the actual data transfer and only pass the Global Transaction ID to the receiving node. That is, whether the node should initiate an :term:`Incremental State Transfer`.



.. _`mysqldump-sst-parameters`:
.. rst-class:: sub-heading
.. rubric:: Logical State Transfer-specific Parameters

These parameters are passed only to the ``wsrep_sst_mysqldump`` state transfer script by both the sending and receiving nodes:

.. rst-class:: verbose-list

   ``--user`` The node gives to the script the database user, which the script then uses to connect to both donor and joiner database servers. Meaning, this user must be the same on both servers, as defined by the :ref:`wsrep_sst_auth <wsrep_sst_auth>` parameter.

   ``--password`` The node gives to the script the password for the database user, as configured by the :ref:`wsrep_sst_auth <wsrep_sst_auth>` parameter.

   ``--host`` The node gives to the script the IP address of the joiner node.

   ``--port`` The node gives to the script the port number to use with the joiner node.

   ``--local-port`` The node gives to the script the port number to use in sending the state transfer.


.. _`calling-conventions`:
.. rst-class:: section-heading
.. rubric:: Calling Conventions

In writing your own custom script for state snapshot transfers, there are certain conventions that you need to follow in order to accommodate how Galera Cluster calls the script.


.. _`call-receiver`:
.. rst-class:: sub-heading
.. rubric:: Receiver

When the node calls for a state snapshot transfer as a joiner, it begins by passing a number of arguments to the state transfer script, as defined in :ref:`General Parameters <general-sst-script-parameters>` above. For your own script you can choose to use or ignore these arguments as suits your needs.

After the script receives these arguments, prepare the node to accept a state snapshot transfer. For example, in the case of ``wsrep_sst_rsync``, the script starts ``rsync`` in server mode.

To signal that the node is ready to receive the state transfer, print the following string to standard output: ``ready <address>:port\n``. Use the IP address and port at which the node is waiting for the state snapshot. For example:

.. code-block:: console

   ready 192.168.1.1:4444

The node responds by sending a state transfer request to the donor node. The node forms the request with the address and port number of the joiner node, the values given to :ref:`wsrep_sst_auth <wsrep_sst_auth>`, and the name of your script. The donor receives the request and uses these values as input parameters in running your script on that node to send back the state transfer.

When the joiner node receives the state transfer and finishes applying it, print to standard output the :term:`Global Transaction ID` of the received state. For example:

.. code-block:: console

	e2c9a15e-5485-11e0-0800-6bbb637e7211:8823450456

Then exit the script with a ``0`` status, to indicate that the state transfer was successful.


.. _`call-sender`:
.. rst-class:: sub-heading
.. rubric:: Sender

When the node calls for a state snapshot transfer as a donor, it begins by passing a number of arguments to the state transfer script, as defined in :ref:`General Parameters <general-sst-script-parameters>` above. For your own script, you can choose to use or ignore these arguments as suits your needs.

While your script runs, Galera Cluster accepts the following signals. You can trigger them by printing to standard output:

.. rst-class:: verbose-list

   ``flush tables\n`` Optional signal that asks the database server to run ``FLUSH TABLES``. When complete, the database server creates a ``tables_flushed`` file in the data directory.

   ``continue\n`` Optional signal that tells the database server that it can continue to commit transactions.

   
   Progress reporting is also enabled for the ``clone`` SST method.

   ``done\n`` Mandatory signal that tells the database server that the state transfer is complete and successful.

   After your script sends the ``done\n`` signal, exit with a ``0`` return code.

In the event of failure, Galera Cluster expects your script to return a code that corresponds to the error it encountered. The donor node returns this code to the joiner through group communication. Given that its data directory now holds an inconsistent state, the joiner node then leaves the cluster and aborts the state transfer.

.. note:: Without the ``continue\n`` signal, your script runs in Total Order Isolation, which guarantees that no further commits occur until the script exits.

The script outputs control messages to the standard output, from where they are read by the parent ``mysqld`` process. These are:

- ``total\n`` This progress reporting parameter indicates the new SST stage and repor-ts the estimated total work.

- ``complete\n`` This progress reporting parameter reports the work completed so far.


.. _`enabling-ssst`:
.. rst-class:: section-heading
.. rubric:: Enabling Scriptable SST's

Whether you use ``wsrep_sst_common`` directly or decide to write a script of your own from scratch, the process for enabling it remains the same. The filename must follow the convention of ``wsrep_sst_<name>``, with ``<name>`` being the value that you give for the :ref:`wsrep_sst_method <wsrep_sst_method>` parameter in the configuration file.

For example, if you write a script with the filename ``wsrep_sst_galera-sst``, you would add the following line to your ``my.cnf``:

.. code-block:: ini

   wsrep_sst_method = galera-sst

When the node starts, it uses your custom script for state snapshot transfers.

.. container:: bottom-links

   Related Documents

   - :ref:`wsrep_sst_receive_address <wsrep_sst_receive_address>`
   - :ref:`wsrep_sst_auth <wsrep_sst_auth>`
   - :ref:`wsrep_sst_method <wsrep_sst_method>`

.. |---|   unicode:: U+2014 .. EM DASH
   :trim:

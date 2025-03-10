.. meta::
   :title: Galera Cluster Glossary
   :description:
   :language: en-US
   :keywords: galera cluster, mysql, mariadb, glossary, terms
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
.. _`glossary`:

==========
 Glossary
==========

.. glossary::
   :sorted:


   Galera Arbitrator
      An external process that functions as an additional node in certain cluster operations, such as :term:`Quorum` calculations and generating consistent application state snapshots.

      For example, consider a situation where your cluster becomes partitioned due to a loss of network connectivity that results in two components of equal size. Each component initiates quorum calculations to determine which should remain the :term:`Primary Component` and which should become a non-operational component. If the components are of equal size, it risks a split-brain condition. Galera Arbitrator provides an addition vote in the quorum calculation, so that one component registers as larger than the other. The larger component then remains the Primary Component.

      Unlike the main ``mysqld`` process, ``garbd`` does not generate replication events of its own and does not store replication data. It does, however, acknowledge all replication events. Furthermore, you can route replication through Galera Arbitrator, such as when generating a consistent application state snapshot for backups.

      For more information, see :doc:`arbitrator` and :doc:`backup-cluster`.


   Galera Replication Plugin
      Galera Replication Plugin is a general purpose replication plugin for any transactional system. It can be used to create a synchronous multi-primary replication solution to achieve high availability and scale-out.

      See :ref:`Galera Replication Plugin <galera-replication-plugin>` for more details.

   Global Transaction ID
      To keep the state identical on all nodes, the :term:`wsrep API` uses global transaction IDs (GTID), which are used to identify the state change and to identify the state itself by the ID of the last state change

      The GTID consists of a state UUID, which uniquely identifies the state and the sequence of changes it undergoes, and an ordinal sequence number (seqno, a 64-bit signed integer) to denote the position of the change in the sequence.

      For more information on Global Transaction ID's, see :ref:`wsrep API <wsrep-api>`.


   Incremental State Transfer
      In an Incremental State Transfer (IST) a node only receives the missing write-sets and catches up with the group by replaying them. See also the definition for State Snapshot Transfer (SST).

      For more information on IST's, see :ref:`Incremental State Transfer (IST) <state-transfer-ist>`.

   IST
      See :term:`Incremental State Transfer`.

   Logical State Transfer Method
      This is a type of back-end state transfer method that operates through the database server (for example, ``mysqldump``).

      For more information, see :ref:`Logical State Snapshot <sst-logical>`.
	  
   Non-Blocking Operations
      With the NBO method, :abbr:`DDL (Data Definition Language)` statements are processed similarly as with the :abbr:`TOI (Total Order Isolation)` method, except that the NBO method uses a more efficient locking strategy. Compared with TOI, NBO offers significant flexibility in DDL statement processing.

      For more information, see :ref:`Non-Blocking Operations <nbo>`.

   NBO
      See :term:`Non-Blocking Operations`.

   PF
      PF is packet filter used for firewall software, that has been ported to several operating systems. It can be configured for firewall protection of a Galera Cluster.

   Physical State Transfer Method
      This is another type of back-end state transfer method, but it operates on the physical media in the datadir (for example, ``rsync`` and ``xtrabackup``).

      For more information, see :ref:`Physical State Snapshot <sst-physical>`.

   Primary Component
      In addition to single-node failures, the cluster may be split into several components due to network failure. In such a situation, only one of the components can continue to modify the database state to avoid history divergence. This component is called the Primary Component (PC).

      For more information on the Primary Component, see :doc:`weighted-quorum`.

   Rolling Schema Upgrade
      The rolling schema upgrade is a :abbr:`DDL (Data Definition Language)` processing method in which the :abbr:`DDL (Data Definition Language)` will only be processed locally on the node. The node is desynchronized from the cluster for the duration of the :abbr:`DDL (Data Definition Language)` processing in a way that it does not block the other nodes. When the :abbr:`DDL (Data Definition Language)` processing is complete, the node applies the delayed replication events and synchronizes with the cluster.

      For more information, see :ref:`Rolling Schema Upgrade <rsu>`.

   RSU
      See :term:`Rolling Schema Upgrade`.

   seqno
      See :term:`Sequence Number`.

   Sequence Number
      This is a 64-bit signed integer that the node uses to denote the position of a given transaction in the sequence. The seqno is second component to the :term:`Global Transaction ID`.

   State Snapshot Transfer
      State Snapshot Transfer refers to a full data copy from one cluster node (that is, a donor) to the joining node (that is, a joiner). See also the definition for Incremental State Transfer (IST).

      For more information, see :ref:`State Snapshot Transfer (SST) <sst>`.

   State UUID
      Unique identifier for the state of a node and the sequence of changes it undergoes. It is the first component of the :term:`Global Transaction ID`.

   SST
      See :term:`State Snapshot Transfer`.


   Streaming Replication
      This provides an alternative replication method for handling large or long-running write transactions. It is a new feature in version 4.0 of Galera Cluster. In older versions, the feature is unsupported.

      Under normal operation, the node performs all replication and certification operations when the transaction commits. With large transactions this can result in conflicts if smaller transactions are committed first. With Streaming Replication, the node breaks the transaction into fragments, then certifies and replicates them to all nodes while the transaction is still in progress. Once certified, a fragment can no longer be aborted by a conflicting transaction.

      For more information see :doc:`streaming-replication` and :doc:`using-sr`.


   Total Order Isolation
      By default, :abbr:`DDL (Data Definition Language)` statements are processed by using the Total Order Isolation (TOI) method. In TOI, the query is replicated to the nodes in a statement form before executing on the primary. The query waits for all preceding transactions to commit and then gets executed in isolation on all nodes, simultaneously.

      For more information, see :ref:`Total Order Isolation <toi>`.

   TOI
      See :term:`Total Order Isolation`.

   write-set
      Transaction commits the node sends to and receives from the cluster.

   Write-set Cache
      Galera stores write-sets in a special cache called, Write-set Cache (GCache). GCache is a memory allocator for write-sets. Its primary purpose is to minimize the write set footprint on the RAM.

      For more information, see :ref:`Write-Set Cache (GCache) <state-transfer-gcache>`.

   GCache
      See :term:`Write-set Cache`.

   wsrep API
      The wsrep API is a generic replication plugin interface for databases. The API defines a set of application callbacks and replication plugin calls.

      For more information, see :ref:`wsrep API <wsrep-api>`.

   Cluster Replication
      Normal replication path for cluster members. Can be encrypted (not by default) and unicast or multicast (unicast by default). Runs on tcp port 4567 by default.
	  
   Donor Node
      The node elected to provide a state transfer (:term:`SST` or :term:`IST`).
	  
   Joiner Node
      The node joining the cluster, usually a state transfer target.
	  
   Node
      A cluster node – a single mysql instance that is in the cluster.
	  
   Primary Cluster
      A cluster with quorum. A non-primary cluster will not allow any operations and will give ``Unknown command`` errors on any clients attempting to read or write from the database.
	  
   Quorum
      A majority (> 50%) of nodes. In the event of a network partition, only the cluster partition that retains a quorum (if any) will remain Primary by default.
	  
   Split Brain
      Split brain occurs when two parts of a computer cluster are disconnected, each part believing that the other is no longer running. This problem can lead to data inconsistency.
.. meta::
   :title: Recovering a Primary Component
   :description:
   :language: en-US
   :keywords: galera cluster, primary component recovery
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

      - :ref:`pc.recovery <pc.recovery>`

.. container:: top-links

   - `Home <https://galeracluster.com>`_

   .. cssclass:: here

      - :doc:`Docs <./index>`

   - :doc:`KB <../kb/index>`

   .. cssclass:: nav-wider

      - :doc:`Training <../training/index>`

   - :doc:`FAQ <../faq>`


.. cssclass:: library-document
.. _`pc-recovery`:

=================================
Recovering Primary Component
=================================

.. index::
   pair: Configuration; pc.recovery


Cluster nodes can store the :term:`Primary Component` state to disk. The node records the state of the Primary Component and the UUID's of the nodes connected to it. In the event of an outage, once all nodes that were part of the last saved state achieve connectivity, the cluster recovers the Primary Component.

If the write-set position differs between the nodes, the recovery process also requires a full state snapshot transfer.

For more information on this feature, see the :ref:`pc.recovery <pc.recovery>` parameter. By default, it is enabled as of version 3.6.


.. _`understand-pc-state`:
.. rst-class:: section-heading
.. rubric:: Understanding the Primary Component State

When a node stores the :term:`Primary Component` state to disk, it saves it as the ``gvwstate.dat`` file. You'll find this file in the database data directory on the server which is acting as the Primary Component.

The node creates and updates this file when the cluster forms or changes the Primary Component. This ensures that the node retains the latest Primary Component state that it was in. If the node loses connectivity, it has the file to reference.

If the node shuts down gracefully, it deletes the file. If the cluster continues after the node has shutdown (that is, there are other nodes that did not shutdown), one of the remaining nodes will become the host to the Primary Component and will create the ``gvwstate.dat`` file on its file system.

Below is an example of the contents of the ``gvwstate.dat`` file:

.. code-block:: text

   my_uuid: d3124bc8-1605-11e4-aa3d-ab44303c044a
   #vwbeg
   view_id: 3 0dae1307-1606-11e4-aa94-5255b1455aa0 12
   bootstrap: 0
   member: 0dae1307-1606-11e4-aa94-5255b1455aa0 1
   member: 47bbe2e2-1606-11e4-8593-2a6d8335bc79 1
   member: d3124bc8-1605-11e4-aa3d-ab44303c044a 1
   #vwend

The ``gvwstate.dat`` file is composed of two parts. **Node Information** provides the node's UUID, in the ``my_uuid`` field. **View Information** provides information on the node's view of the Primary Component, contained between the ``#vwbeg`` and ``#vwend`` tags.

The ``view_id`` forms an identifier for the view from three parts: *view_type*, which always gives a value of ``3`` to indicate the primary view; and the *view_uuid* and *view_seq* together form a unique value for the identifier.

The ``bootstrap`` variable indicates whether or not the node is bootstrapped. It does not, though, effect the Primary Component recovery process. The ``member`` variables contain the UUID's of nodes connecting to the Primary Component.


.. _`modifying-pc-state`:
.. rst-class:: section-heading
.. rubric:: Modifying the Saved Primary Component State

If you find yourself in the unusual situation where you need to force certain nodes to join each other specifically, you can do so by manually changing the saved :term:`Primary Component` state.

.. warning:: Under normal circumstances, for safety reasons, you should entirely avoid editing or otherwise modifying the ``gvwstate.dat`` file. Doing so may lead to unexpected results.

When a node starts for the first time or after a graceful shutdown, it randomly generates and assigns to itself a UUID, which serves as its identifier to the rest of the cluster. If the node finds a ``gvwstate.dat`` file in the data directory, it reads the ``my_uuid`` field to find the value it should use.

By manually assigning arbitrary UUID values to the respective fields on each node, you force them to join each other, forming a new Primary Component, as they start.

For example, assume that you have three nodes that you would like to start together to form a new Primary Component for the cluster. You will need to generate three UUID values, one for each node.

.. code-block:: mysql

   SELECT UUID();

   +--------------------------------------+
   | UUID()                               |
   +--------------------------------------+
   | 47bbe2e2-1606-11e4-8593-2a6d8335bc79 |
   +--------------------------------------+

You would then take these values and use them to modify the ``gwstate.dat`` file on ``node1``:

.. code-block:: text

   my_uuid: d3124bc8-1605-11e4-aa3d-ab44303c044a
   #vwbeg
   view_id: 3 0dae1307-1606-11e4-aa94-5255b1455aa0 12
   bootstrap: 0
   member: 0dae1307-1606-11e4-aa94-5255b1455aa0 1
   member: 47bbe2e2-1606-11e4-8593-2a6d8335bc79 1
   member: d3124bc8-1605-11e4-aa3d-ab44303c044a 1
   #vwend

Then repeat the process for ``node2``:

.. code-block:: text

   my_uuid: 47bbe2e2-1606-11e4-8593-2a6d8335bc79
   #vwbeg
   view_id: 3 0dae1307-1606-11e4-aa94-5255b1455aa0 12
   bootstrap: 0
   member: 0dae1307-1606-11e4-aa94-5255b1455aa0 1
   member: 47bbe2e2-1606-11e4-8593-2a6d8335bc79 1
   member: d3124bc8-1605-11e4-aa3d-ab44303c044a 1
   #vwend

And, the same again for ``node3``:

.. code-block:: text

   my_uuid: d3124bc8-1605-11e4-aa3d-ab44303c044a
   #vwbeg
   view_id: 3 0dae1307-1606-11e4-aa94-5255b1455aa0 12
   bootstrap: 0
   member: 0dae1307-1606-11e4-aa94-5255b1455aa0 1
   member: 47bbe2e2-1606-11e4-8593-2a6d8335bc79 1
   member: d3124bc8-1605-11e4-aa3d-ab44303c044a 1
   #vwend


Then start all three nodes without the bootstrap flag. When they start, Galera Cluster reads the ``gvwstate.dat`` file for each. It pulls its UUID from the file and uses those of the ``member`` field to determine which nodes it should join in order to form a new Primary Component.

.. container:: bottom-links

   Related Documents

   - :ref:`pc.recovery <pc.recovery>`

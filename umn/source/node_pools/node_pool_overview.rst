:original_name: cce_01_0081.html

.. _cce_01_0081:

Node Pool Overview
==================

Introduction
------------

CCE introduces node pools to help you better manage nodes in Kubernetes clusters. A node pool contains one node or a group of nodes with identical configuration in a cluster.

You can create custom node pools on the CCE console. With node pools, you can quickly create, manage, and destroy nodes without affecting the cluster. All nodes in a custom node pool have identical parameters and node type. You cannot configure a single node in a node pool; any configuration changes affect all nodes in the node pool.

You can also use node pools for auto scaling.

-  When a pod in a cluster cannot be scheduled due to insufficient resources, scale-out can be automatically triggered.
-  When there is an idle node or a monitoring metric threshold is met, scale-in can be automatically triggered.

This section describes how node pools work in CCE and how to create and manage node pools.

Node Pool Architecture
----------------------


.. figure:: /_static/images/en-us_image_0269288708.png
   :alt: **Figure 1** Overall architecture of a node pool

   **Figure 1** Overall architecture of a node pool

Generally, all nodes in a node pool have the following same attributes:

-  Node OS
-  Startup parameters of Kubernetes components on a node
-  User-defined startup script of a node
-  **K8S Labels** and **Taints**

CCE provides the following extended attributes for node pools:

-  Node pool OS
-  Maximum number of pods on each node in a node pool

Description of DefaultPool
--------------------------

DefaultPool is not a real node pool. It only **classifies** nodes that are not in any node pool. These nodes are directly created on the console or by calling APIs. DefaultPool does not support any node pool functions, including scaling and parameter configuration. DefaultPool cannot be edited, deleted, expanded, or auto scaled, and nodes in it cannot be migrated.

Applicable Scenarios
--------------------

When a large-scale cluster is required, you are advised to use node pools to manage nodes.

The following table describes multiple scenarios of large-scale cluster management and the functions of node pools in each scenario.

.. table:: **Table 1** Using node pools for different management scenarios

   +----------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
   | Scenario                                                                               | Function                                                                |
   +========================================================================================+=========================================================================+
   | Multiple heterogeneous nodes (with different models and configurations) in the cluster | Nodes can be grouped into different pools for management.               |
   +----------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
   | Frequent node scaling required in a cluster                                            | Node pools support auto scaling to dynamically add or reduce nodes.     |
   +----------------------------------------------------------------------------------------+-------------------------------------------------------------------------+
   | Complex application scheduling rules in a cluster                                      | Node pool tags can be used to quickly specify service scheduling rules. |
   +----------------------------------------------------------------------------------------+-------------------------------------------------------------------------+

Functions and Precautions
-------------------------

+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Function                               | Description                                                                                                                                            | Notes                                                                                                                                                                              |
+========================================+========================================================================================================================================================+====================================================================================================================================================================================+
| Creating a node pool                   | Add a node pool.                                                                                                                                       | It is recommended that a cluster contain no more than 100 node pools.                                                                                                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Deleting a node pool                   | Deleting a node pool will delete nodes in the pool. Pods on these nodes will be automatically migrated to available nodes in other node pools.         | If pods in the node pool have a specific node selector and none of the other nodes in the cluster satisfies the node selector, the pods will become unschedulable.                 |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Enabling auto scaling for a node pool  | After auto scaling is enabled, nodes will be automatically created or deleted in the node pool based on the cluster loads.                             | You are advised not to store important data on nodes in a node pool because after auto scaling, data cannot be restored as nodes may be deleted.                                   |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Enabling auto scaling for a node pool  | After auto scaling is disabled, the number of nodes in a node pool will not automatically change with the cluster loads.                               | /                                                                                                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Adjusting the size of a node pool      | The number of nodes in a node pool can be directly adjusted. If the number of nodes is reduced, nodes are randomly removed from the current node pool. | After auto scaling is enabled, you are not advised to manually adjust the node pool size.                                                                                          |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Changing node pool configurations      | You can modify the node pool name, node quantity, Kubernetes labels, taints, and resource tags.                                                        | The modified Kubernetes labels and taints will apply to all nodes in the node pool, which may cause pod re-scheduling. Therefore, exercise caution when performing this operation. |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Adding an existing node to a node pool | Nodes that do not belong to the cluster can be added to a node pool. The following requirements must be met:                                           | Unless required, you are not advised to add existing nodes. You are advised to create a node pool.                                                                                 |
|                                        |                                                                                                                                                        |                                                                                                                                                                                    |
|                                        | -  The node to be added and the CCE cluster are in the same VPC and subnet.                                                                            |                                                                                                                                                                                    |
|                                        | -  The node is not used by other clusters and has the same configurations (such as specifications and billing mode) as the node pool.                  |                                                                                                                                                                                    |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Removing a node from a node pool       | Nodes in a node pool can be migrated to the default node pool of the same cluster.                                                                     | Nodes in the default node pool cannot be migrated to other node pools, and nodes in a user-created node pool cannot be migrated to other user-created node pools.                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Cloning a node pool                    | You can copy the configuration of an existing node pool to create a new node pool.                                                                     | /                                                                                                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Setting Kubernetes parameters          | You can configure core components with fine granularity.                                                                                               | -  This function is supported only for clusters of v1.15 and later. It is not displayed for versions earlier than v1.15                                                            |
|                                        |                                                                                                                                                        | -  The default node pool DefaultPool does not support this type of configuration.                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Deploying a Workload in a Specified Node Pool
---------------------------------------------

When creating a workload, you can constrain pods to run in a specified node pool.

For example, on the CCE console, you can set the affinity between the workload and the node on the **Scheduling Policies** tab page on the workload details page to forcibly deploy the workload to a specific node pool. In this way, the workload runs only on nodes in the node pool. If you need to better control where the workload is to be scheduled, you can use affinity or anti-affinity policies between workloads and nodes described in :ref:`Scheduling Policy Overview <cce_01_0051>`.

For example, you can use container's resource request as a nodeSelector so that workloads will run only on the nodes that meet the resource request.

If the workload definition file defines a container that requires four CPUs, the scheduler will not choose the nodes with two CPUs to run workloads.

Related Operations
------------------

You can log in to the CCE console and refer to the following sections to perform operations on node pools:

-  :ref:`Creating a Node Pool <cce_01_0012>`
-  :ref:`Managing a Node Pool <cce_01_0222>`
-  :ref:`Creating a Deployment <cce_01_0047>`
-  :ref:`Workload-Node Affinity <cce_01_0225>`

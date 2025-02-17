:original_name: cce_01_0252.html

.. _cce_01_0252:

Using kubectl to Create an ELB Ingress
======================================

Scenario
--------

This section uses an :ref:`Nginx workload <cce_01_0047__section155246177178>` as an example to describe how to create an ELB ingress using kubectl.

-  If no load balancer is available in the same VPC, CCE can automatically create a load balancer when creating an ingress. For details, see :ref:`Creating an Ingress - Automatically Creating a Load Balancer <cce_01_0252__section3675115714214>`.
-  If a load balancer is available in the same VPC, perform the operation by referring to :ref:`Creating an Ingress - Interconnecting with an Existing Load Balancer <cce_01_0252__section32300431736>`.

Prerequisites
-------------

-  An ingress provides network access for backend workloads. Ensure that a workload is available in a cluster. If no workload is available, deploy a sample Nginx workload by referring to :ref:`Creating a Deployment <cce_01_0047>`, :ref:`Creating a StatefulSet <cce_01_0048>`, or :ref:`Creating a DaemonSet <cce_01_0216>`.
-  A NodePort Service has been configured for the workload. For details about how to configure the Service, see :ref:`NodePort <cce_01_0142>`.

.. _cce_01_0252__section3675115714214:

Creating an Ingress - Automatically Creating a Load Balancer
------------------------------------------------------------

The following describes how to run the kubectl command to automatically create a load balancer when creating an ingress.

#. Use kubectl to connect to the cluster. For details, see :ref:`Connecting to a Cluster Using kubectl <cce_01_0107>`.

#. Create a YAML file named **ingress-test.yaml**. The file name can be customized.

   **vi ingress-test.yaml**

   .. note::

      -  For clusters of v1.15 or later, the value of **apiVersion** is **networking.k8s.io/v1beta1**.
      -  For clusters of v1.13 or earlier, the value of **apiVersion** is **extensions/v1beta1**.

   You can create a load balancer as required. The YAML files are as follows:

   **Example of using a dedicated public network load balancer**:

   .. code-block::

      apiVersion: networking.k8s.io/v1beta1
      kind: Ingress
      metadata:
        name: ingress-test
        annotations:
          kubernetes.io/elb.class: union
          kubernetes.io/ingress.class: cce
          kubernetes.io/elb.port: '80'
          kubernetes.io/elb.autocreate:
            '{
                "type":"public",
                "bandwidth_name":"cce-bandwidth-******",
                "bandwidth_chargemode":"traffic",
                "bandwidth_size":5,
                "bandwidth_sharetype":"PER",
                "eip_type":"5_bgp"
              }'
      spec:
        rules:
        - host: ''
          http:
            paths:
            - path: '/'
              backend:
                serviceName: <your_service_name>  # Replace it with the name of your target Service.
                servicePort: 80
              property:
                ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

   **Example of using a dedicated public network load balancer:**

   .. code-block::

      apiVersion: networking.k8s.io/v1beta1
      kind: Ingress
      metadata:
        name: ingress-test
        namespace: default
        annotations:
          kubernetes.io/elb.class: performance
          kubernetes.io/ingress.class: cce
          kubernetes.io/elb.port: '80'
          kubernetes.io/elb.autocreate:
            '{
                "type": "public",
                "bandwidth_name": "cce-bandwidth-******",
                "bandwidth_chargemode": "traffic",
                "bandwidth_size": 5,
                "bandwidth_sharetype": "PER",
                "eip_type": "5_bgp",
                "available_zone": [
                    "eu-de-01"
                ],
                "l7_flavor_name": "L7_flavor.elb.s1.small"
             }'
      spec:
        rules:
        - host: ''
          http:
            paths:
            - path: '/'
              backend:
                serviceName: <your_service_name>  # Replace it with the name of your target Service.
                servicePort: 80
              property:
                ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

   .. table:: **Table 1** Key parameters

      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | Parameter                                 | Mandatory       | Type                  | Description                                                                                                                                                                                                                              |
      +===========================================+=================+=======================+==========================================================================================================================================================================================================================================+
      | kubernetes.io/elb.class                   | No              | String                | Select a proper load balancer type.                                                                                                                                                                                                      |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | The value can be:                                                                                                                                                                                                                        |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | -  **union**: shared load balancer                                                                                                                                                                                                       |
      |                                           |                 |                       | -  **performance**: dedicated load balancer..                                                                                                                                                                                            |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | The default value is **union**.                                                                                                                                                                                                          |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | kubernetes.io/ingress.class               | Yes             | String                | **cce**: The self-developed ELBIngress is used.                                                                                                                                                                                          |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | This parameter is mandatory when an ingress is created by calling the API.                                                                                                                                                               |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | kubernetes.io/elb.port                    | Yes             | Integer               | This parameter indicates the external port registered with the address of the LoadBalancer Service.                                                                                                                                      |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | Supported range: 1 to 65535                                                                                                                                                                                                              |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | kubernetes.io/elb.subnet-id               | -               | String                | ID of the subnet where the cluster is located. The value can contain 1 to 100 characters.                                                                                                                                                |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | -  Mandatory when a cluster of v1.11.7-r0 or earlier is to be automatically created.                                                                                                                                                     |
      |                                           |                 |                       | -  Optional for clusters later than v1.11.7-r0. It is left blank by default.                                                                                                                                                             |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | kubernetes.io/elb.enterpriseID            | No              | String                | **Kubernetes clusters of v1.15 and later versions support this field. In Kubernetes clusters earlier than v1.15, load balancers are created in the default project by default.**                                                         |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | ID of the enterprise project in which the load balancer will be created.                                                                                                                                                                 |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | The value contains 1 to 100 characters.                                                                                                                                                                                                  |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | **How to obtain**:                                                                                                                                                                                                                       |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | Log in to the management console and choose **Enterprise** > **Project Management** on the top menu bar. In the list displayed, click the name of the target enterprise project, and copy the ID on the enterprise project details page. |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | kubernetes.io/elb.autocreate              | Yes             | elb.autocreate object | Whether to automatically create a load balancer associated with an ingress. For details about the field description, see :ref:`Table 2 <cce_01_0252__table268711532210>`.                                                                |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | **Example**                                                                                                                                                                                                                              |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | -  If a public network load balancer will be automatically created, set this parameter to the following value:                                                                                                                           |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       |    '{"type":"public","bandwidth_name":"cce-bandwidth-******","bandwidth_chargemode":"traffic","bandwidth_size":5,"bandwidth_sharetype":"PER","eip_type":"5_bgp","name":"james"}'                                                         |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | -  If a private network load balancer will be automatically created, set this parameter to the following value:                                                                                                                          |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       |    {"type":"inner","name":"A-location-d-test"}                                                                                                                                                                                           |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | host                                      | No              | String                | Domain name for accessing the Service. By default, this parameter is left blank, and the domain name needs to be fully matched.                                                                                                          |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | path                                      | Yes             | String                | User-defined route path. All external access requests must match **host** and **path**.                                                                                                                                                  |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | serviceName                               | Yes             | String                | Name of the target Service bound to the ingress.                                                                                                                                                                                         |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | servicePort                               | Yes             | Integer               | Access port of the target Service.                                                                                                                                                                                                       |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | ingress.beta.kubernetes.io/url-match-mode | No              | String                | Route matching policy.                                                                                                                                                                                                                   |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | Default: **STARTS_WITH** (prefix match)                                                                                                                                                                                                  |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | Options:                                                                                                                                                                                                                                 |
      |                                           |                 |                       |                                                                                                                                                                                                                                          |
      |                                           |                 |                       | -  **EQUAL_TO**: exact match                                                                                                                                                                                                             |
      |                                           |                 |                       | -  **STARTS_WITH**: prefix match                                                                                                                                                                                                         |
      |                                           |                 |                       | -  **REGEX**: regular expression match                                                                                                                                                                                                   |
      +-------------------------------------------+-----------------+-----------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

   .. _cce_01_0252__table268711532210:

   .. table:: **Table 2** Data structure of the elb.autocreate field

      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | Parameter            | Mandatory                             | Type            | Description                                                                                                                                                                                                                        |
      +======================+=======================================+=================+====================================================================================================================================================================================================================================+
      | type                 | No                                    | String          | Network type of the load balancer.                                                                                                                                                                                                 |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | -  **public**: public network load balancer                                                                                                                                                                                        |
      |                      |                                       |                 | -  **inner**: private network load balancer                                                                                                                                                                                        |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | The default value is **inner**.                                                                                                                                                                                                    |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | bandwidth_name       | Yes for public network load balancers | String          | Bandwidth name. The default value is **cce-bandwidth-*****\***.                                                                                                                                                                    |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | Value range: a string of 1 to 64 characters, including lowercase letters, digits, and underscores (_). The value must start with a lowercase letter and end with a lowercase letter or digit.                                      |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | bandwidth_chargemode | Yes                                   | String          | Bandwidth billing mode.                                                                                                                                                                                                            |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | -  **traffic**: billed by traffic                                                                                                                                                                                                  |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | bandwidth_size       | Yes for public network load balancers | Integer         | Bandwidth size. The value ranges from 1 Mbit/s to 2000 Mbit/s by default. The actual range varies depending on the configuration in each region.                                                                                   |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | -  The minimum increment for bandwidth adjustment varies depending on the bandwidth range. The details are as follows:                                                                                                             |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 |    -  The minimum increment is 1 Mbit/s if the allowed bandwidth ranges from 0 Mbit/s to 300 Mbit/s (with 300 Mbit/s included).                                                                                                    |
      |                      |                                       |                 |    -  The minimum increment is 50 Mbit/s if the allowed bandwidth ranges from 300 Mbit/s to 1000 Mbit/s.                                                                                                                           |
      |                      |                                       |                 |    -  The minimum increment is 500 Mbit/s if the allowed bandwidth is greater than 1000 Mbit/s.                                                                                                                                    |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | bandwidth_sharetype  | Yes for public network load balancers | String          | Bandwidth type.                                                                                                                                                                                                                    |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | **PER**: dedicated bandwidth                                                                                                                                                                                                       |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | eip_type             | Yes for public network load balancers | String          | EIP type, which may vary depending on sites. For details, see the type parameter specified when `creating an EIP <https://docs.otc.t-systems.com/api/eip/eip_api_0001.html#eip_api_0001__en-us_topic_0201534274_table44471219>`__. |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | -  **5_bgp**: dynamic BGP                                                                                                                                                                                                          |
      |                      |                                       |                 | -  **5_gray**: dedicated load balancer                                                                                                                                                                                             |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | name                 | No                                    | String          | Name of the automatically created load balancer.                                                                                                                                                                                   |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | Value range: a string of 1 to 64 characters, including lowercase letters, digits, and underscores (_). The value must start with a lowercase letter and end with a lowercase letter or digit.                                      |
      |                      |                                       |                 |                                                                                                                                                                                                                                    |
      |                      |                                       |                 | Default value: **cce-lb+ingress.UID**                                                                                                                                                                                              |
      +----------------------+---------------------------------------+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

#. Create an ingress.

   **kubectl create -f ingress-test.yaml**

   If information similar to the following is displayed, the ingress has been created.

   .. code-block::

      ingress/ingress-test created

   **kubectl get ingress**

   If information similar to the following is displayed, the ingress has been created successfully and the workload is accessible.

   .. code-block::

      NAME             HOSTS     ADDRESS          PORTS   AGE
      ingress-test     *         121.**.**.**     80      10s

#. Enter **http://121.**.**.*\*:80** in the address box of the browser to access the workload (for example, :ref:`Nginx workload <cce_01_0047__section155246177178>`).

   **121.**.**.*\*** indicates the IP address of the unified load balancer.

.. _cce_01_0252__section32300431736:

Creating an Ingress - Interconnecting with an Existing Load Balancer
--------------------------------------------------------------------

CCE allows you to connect to an existing load balancer when creating an ingress.

.. note::

   -  For clusters of v1.15 or later, the value of **apiVersion** is **networking.k8s.io/v1beta1**.
   -  For clusters of v1.13 or earlier, the value of **apiVersion** is **extensions/v1beta1**.
   -  To interconnect with an existing dedicated load balancer, ensure that HTTP is supported and the network type supports private networks.

**If the cluster version is 1.15 or later, the YAML file configuration is as follows:**

.. code-block::

   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
     name: ingress-test
     annotations:
       kubernetes.io/elb.class: performance                               # Load balancer type
       kubernetes.io/elb.id: <your_elb_id>  # Replace it with the ID of your existing load balancer.
       kubernetes.io/elb.ip: <your_elb_ip>  # Replace it with your existing load balancer IP.
       kubernetes.io/elb.port: '80'
       kubernetes.io/ingress.class: cce
   spec:
     rules:
     - host: ''
       http:
         paths:
         - path: '/'
           backend:
             serviceName: <your_service_name>  # Replace it with the name of your target Service.
             servicePort: 80
           property:
             ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

.. table:: **Table 3** Key parameters

   +-------------------------+-----------------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | Parameter               | Mandatory       | Type            | Description                                                                                                                                                                                             |
   +=========================+=================+=================+=========================================================================================================================================================================================================+
   | kubernetes.io/elb.class | No              | String          | Select a proper load balancer type.                                                                                                                                                                     |
   |                         |                 |                 |                                                                                                                                                                                                         |
   |                         |                 |                 | The value can be:                                                                                                                                                                                       |
   |                         |                 |                 |                                                                                                                                                                                                         |
   |                         |                 |                 | -  **union**: shared load balancer                                                                                                                                                                      |
   |                         |                 |                 | -  **performance**: dedicated load balancer..                                                                                                                                                           |
   |                         |                 |                 |                                                                                                                                                                                                         |
   |                         |                 |                 | Defaults to **union**.                                                                                                                                                                                  |
   +-------------------------+-----------------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | kubernetes.io/elb.id    | Yes             | String          | This parameter indicates the ID of a load balancer. The value can contain 1 to 100 characters.                                                                                                          |
   |                         |                 |                 |                                                                                                                                                                                                         |
   |                         |                 |                 | **How to obtain**:                                                                                                                                                                                      |
   |                         |                 |                 |                                                                                                                                                                                                         |
   |                         |                 |                 | On the management console, click **Service List**, and choose **Networking** > **Elastic Load Balance**. Click the name of the target load balancer. On the **Summary** tab page, find and copy the ID. |
   +-------------------------+-----------------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
   | kubernetes.io/elb.ip    | Yes             | String          | This parameter indicates the service address of a load balancer. The value can be the public IP address of a public network load balancer or the private IP address of a private network load balancer. |
   +-------------------------+-----------------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Configuring HTTPS Certificates
------------------------------

Ingress supports TLS certificate configuration and provides security services in HTTPS mode.

.. note::

   -  If a Service needs to be exposed using HTTPS, you must configure the TLS certificate in the ingress. For details on how to create a secret, see :ref:`Creating a Secret <cce_01_0153>`.
   -  If HTTPS is used for the same port of the same load balancer of multiple ingresses, you must select the same certificate.

#. Use kubectl to connect to the cluster. For details, see :ref:`Connecting to a Cluster Using kubectl <cce_01_0107>`.

#. Run the following command to create a YAML file named **ingress-test-secret.yaml** (the file name can be customized):

   **vi ingress-test-secret.yaml**

   **The YAML file is configured as follows:**

   .. code-block::

      apiVersion: v1
      data:
        tls.crt: LS0******tLS0tCg==
        tls.key: LS0tL******0tLS0K
      kind: Secret
      metadata:
        annotations:
          description: test for ingressTLS secrets
        name: ingress-test-secret
        namespace: default
      type: IngressTLS

   .. note::

      In the preceding information, **tls.crt** and **tls.key** are only examples. Replace them with the actual files. The values of **tls.crt** and **tls.key** are the content encrypted using Base64.

#. Create a secret.

   **kubectl create -f ingress-test-secret.yaml**

   If information similar to the following is displayed, the secret is being created:

   .. code-block::

      secret/ingress-test-secret created

   View the created secrets.

   **kubectl get secrets**

   If information similar to the following is displayed, the secret has been created successfully:

   .. code-block::

      NAME                         TYPE                                  DATA      AGE
      ingress-test-secret          IngressTLS                            2         13s

#. Create a YAML file named **ingress-test.yaml**. The file name can be customized.

   **vi ingress-test.yaml**

   .. note::

      Security policy (kubernetes.io/elb.tls-ciphers-policy) is supported only in clusters of v1.17.11 or later.

   **Example YAML file to associate an existing load balancer:**

   .. code-block::

      apiVersion: networking.k8s.io/v1beta1
      kind: Ingress
      metadata:
        name: ingress-test
        annotations:
          kubernetes.io/elb.class: performance                               # Load balancer type
          kubernetes.io/elb.id: <your_elb_id>  # Replace it with the ID of your existing load balancer.
          kubernetes.io/elb.ip: <your_elb_ip>  # Replace it with the IP of your existing load balancer.
          kubernetes.io/ingress.class: cce
          kubernetes.io/elb.port: '443'
          kubernetes.io/elb.tls-ciphers-policy: tls-1-2
      spec:
        tls:
        - secretName: ingress-test-secret
        rules:
        - host: ''
          http:
            paths:
            - path: '/'
              backend:
                serviceName: <your_service_name>  # Replace it with the name of your target Service.
                servicePort: 80
              property:
                ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

   .. table:: **Table 4** Key parameters

      +--------------------------------------+-----------------+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | Parameter                            | Mandatory       | Type             | Description                                                                                                                                                                                                                                |
      +======================================+=================+==================+============================================================================================================================================================================================================================================+
      | kubernetes.io/elb.tls-ciphers-policy | No              | String           | The default value is **tls-1-2**, which is the security policy used by the listener and takes effect only when the HTTPS protocol is used.                                                                                                 |
      |                                      |                 |                  |                                                                                                                                                                                                                                            |
      |                                      |                 |                  | Options:                                                                                                                                                                                                                                   |
      |                                      |                 |                  |                                                                                                                                                                                                                                            |
      |                                      |                 |                  | -  tls-1-0                                                                                                                                                                                                                                 |
      |                                      |                 |                  | -  tls-1-1                                                                                                                                                                                                                                 |
      |                                      |                 |                  | -  tls-1-2                                                                                                                                                                                                                                 |
      |                                      |                 |                  | -  tls-1-2-strict                                                                                                                                                                                                                          |
      |                                      |                 |                  |                                                                                                                                                                                                                                            |
      |                                      |                 |                  | For details of cipher suites for each security policy, see :ref:`Table 5 <cce_01_0252__table9419191416246>`.                                                                                                                               |
      +--------------------------------------+-----------------+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | tls                                  | No              | Array of strings | This parameter is mandatory if HTTPS is used. Multiple independent domain names and certificates can be added to this parameter. For details, see :ref:`Configuring the Server Name Indication (SNI) <cce_01_0252__section0555194782414>`. |
      +--------------------------------------+-----------------+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | secretName                           | No              | String           | This parameter is mandatory if HTTPS is used. Set this parameter to the name of the created secret.                                                                                                                                        |
      +--------------------------------------+-----------------+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

   .. _cce_01_0252__table9419191416246:

   .. table:: **Table 5** tls_ciphers_policy parameter description

      +-----------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | Security Policy       | TLS Version           | Cipher Suite                                                                                                                                                                                                                                                                                                                                                                                          |
      +=======================+=======================+=======================================================================================================================================================================================================================================================================================================================================================================================================+
      | tls-1-0               | TLS 1.2               | ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:AES128-SHA256:AES256-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:AES128-SHA:AES256-SHA |
      |                       |                       |                                                                                                                                                                                                                                                                                                                                                                                                       |
      |                       | TLS 1.1               |                                                                                                                                                                                                                                                                                                                                                                                                       |
      |                       |                       |                                                                                                                                                                                                                                                                                                                                                                                                       |
      |                       | TLS 1.0               |                                                                                                                                                                                                                                                                                                                                                                                                       |
      +-----------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | tls-1-1               | TLS 1.2               |                                                                                                                                                                                                                                                                                                                                                                                                       |
      |                       |                       |                                                                                                                                                                                                                                                                                                                                                                                                       |
      |                       | TLS 1.1               |                                                                                                                                                                                                                                                                                                                                                                                                       |
      +-----------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | tls-1-2               | TLS 1.2               |                                                                                                                                                                                                                                                                                                                                                                                                       |
      +-----------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
      | tls-1-2-strict        | TLS 1.2               | ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:AES128-SHA256:AES256-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384                                                                                                               |
      +-----------------------+-----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

#. Create an ingress.

   **kubectl create -f ingress-test.yaml**

   If information similar to the following is displayed, the ingress has been created.

   .. code-block::

      ingress/ingress-test created

   View the created ingress.

   **kubectl get ingress**

   If information similar to the following is displayed, the ingress has been created successfully and the workload is accessible.

   .. code-block::

      NAME             HOSTS     ADDRESS          PORTS   AGE
      ingress-test     *         121.**.**.**     80      10s

#. Enter **https://121.**.**.*\*:443** in the address box of the browser to access the workload (for example, :ref:`Nginx workload <cce_01_0047__section155246177178>`).

   **121.**.**.*\*** indicates the IP address of the unified load balancer.

.. _cce_01_0252__section0555194782414:

Configuring the Server Name Indication (SNI)
--------------------------------------------

SNI allows multiple TLS-based access domain names to be provided for external systems using the same IP address and port number. Different domain names can use different security certificates.

.. note::

   -  Only one domain name can be specified for each SNI certificate. Wildcard-domain certificates are supported.
   -  Security policy (kubernetes.io/elb.tls-ciphers-policy) is supported only in clusters of v1.17.11 or later.

You can enable SNI when the preceding conditions are met. The following uses the automatic creation of a load balancer as an example. In this example, **sni-test-secret-1** and **sni-test-secret-2** are SNI certificates. The domain names specified by the certificates must be the same as those in the certificates.

.. code-block::

   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
     name: ingress-test
     annotations:
       kubernetes.io/elb.class: performance                               # Load balancer type
       kubernetes.io/elb.id: <your_elb_id>  # Replace it with the ID of your existing load balancer.
       kubernetes.io/elb.ip: <your_elb_ip>  # Replace it with the IP of your existing load balancer.
       kubernetes.io/ingress.class: cce
       kubernetes.io/elb.port: '443'
       kubernetes.io/elb.tls-ciphers-policy: tls-1-2
   spec:
     tls:
     - secretName: ingress-test-secret
     - hosts:
         - example.top  # Domain name specified a certificate is issued
       secretName: sni-test-secret-1
     - hosts:
         - example.com  # Domain name specified a certificate is issued
       secretName: sni-test-secret-2
     rules:
     - host: ''
       http:
         paths:
         - path: '/'
           backend:
             serviceName: <your_service_name>  # Replace it with the name of your target Service.
             servicePort: 80
           property:
             ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

Accessing Multiple Services
---------------------------

Ingresses can route requests to multiple backend Services based on different matching policies. The **spec** field in the YAML file is set as below. You can access **www.example.com/foo**, **www.example.com/bar**, and **foo.example.com/** to route to three different backend Services.

.. important::

   The URL registered in an ingress forwarding policy must be the same as the URL exposed by the backend Service. Otherwise, a 404 error will be returned.

.. code-block::

   spec:
     rules:
     - host: 'www.example.com'
       http:
         paths:
         - path: '/foo'
           backend:
             serviceName: <your_service_name>  # Replace it with the name of your target Service.
             servicePort: 80
           property:
             ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH
         - path: '/bar'
           backend:
             serviceName: <your_service_name>  # Replace it with the name of your target Service.
             servicePort: 80
           property:
             ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH
     - host: 'foo.example.com'
       http:
         paths:
         - path: '/'
           backend:
             serviceName: <your_service_name>  # Replace it with the name of your target Service.
             servicePort: 80
           property:
             ingress.beta.kubernetes.io/url-match-mode: STARTS_WITH

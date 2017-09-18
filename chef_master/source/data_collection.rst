=====================================================
Configure Data Collection
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/setup_visibility_chef_automate.rst>`__

.. tag chef_automate_mark

.. image:: ../../images/chef_automate_full.png
   :width: 40px
   :height: 17px

.. end_tag

Automatic Node Run Data Collection with Chef Server
=======================================================

.. note:: New in Chef 12.16. Chef 12.16.42 or greater and Chef Server 12.11.0 or greater are required.

Nodes can send their run data to Chef Automate through the Chef server automatically. To enable this functionality, you must perform the following steps:

 * Configure a Data Collector token in Chef Automate
 * Configure your Chef server to point to Chef Automate

Step 1: Configure a Data Collector token in Chef Automate
------------------------------------------------------------

All messages sent to Chef Automate are performed over HTTP and are authenticated with a pre-shared key called a ``token.`` Every Chef Automate installation configures a default token by default, but we strongly recommend that you create your own.

To set your own token, add the following to your ``/etc/delivery/delivery.rb`` file:

   .. code-block:: ruby

      data_collector['token'] = 'sometokenvalue'
      # Save and close the file

To apply the changes, run:

   .. code-block:: ruby

      automate-ctl reconfigure


If you do not configure a token, the default token value is: ``93a49a4f2482c64126f7b6015e6b0f30284287ee4054ff8807fb63d9cbd1c506``

Step 2: Configure your Chef server to point to Chef Automate
-----------------------------------------------------------------
In addition to forwarding Chef run data to Automate, Chef server will send messages to Chef Automate whenever an action is taken on a Chef server object, such as when a cookbook is uploaded to the Chef server or when a user edits a role.

Setting up data collection on Chef server versions 12.14 and higher
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Channel the token setting through the veil secrets library because the token is considered a secret, and cannot appear in ``/etc/opscode/chef-server.rb``:

   .. code-block:: ruby

      chef-server-ctl set-secret data_collector token 'TOKEN'
      chef-server-ctl restart nginx

Then add the following setting to ``/etc/opscode/chef-server.rb`` on the Chef server:

   .. code-block:: ruby

      data_collector['root_url'] = 'https://my-automate-server.mycompany.com/data-collector/v0/'
      # Add for compliance scanning
      profiles['root_url'] = 'https://my-automate-server.mycompany.com'
      # Save and close the file

To apply the changes, run:

   .. code-block:: ruby

      chef-server-ctl reconfigure


where ``my-automate-server.mycompany.com`` is the fully-qualified domain name of your Chef Automate server.

Setting up data collection on Chef server versions 12.13 and lower
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
On versions 12.13 and prior, simply add the ``'root_url'`` and ``token`` values in ``/etc/opscode/chef-server.rb``:

   .. code-block:: ruby

      data_collector['root_url'] = 'https://my-automate-server.mycompany.com/data-collector/v0/'
      data_collector['token'] = 'TOKEN'
      # Add for compliance scanning
      profiles['root_url'] = 'https://my-automate-server.mycompany.com'
      # Save and close the file

To apply the changes, run:

      .. code-block:: ruby

      chef-server-ctl reconfigure


where ``my-automate-server.mycompany.com`` is the fully-qualified domain name of your Chef Automate server, and
``TOKEN`` is either the default value or the token value you configured in the `prior section <#configure-a-data-collector-token-in-chef-automate>`__.

Additional options
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 50 200 100
   :header-rows: 1

   * - Option
     - Description
     - Default
   * - ``data_collector['timeout']``
     - Timeout in milliseconds to abort an attempt to send a message to the Chef Automate server.
     - Default: ``30000``.
   * - ``data_collector['http_init_count']``
     - Number of Chef Automate HTTP workers Chef server should start.
     - Default: ``25``.
   * - ``data_collector['http_max_count']``
     - Maximum number of Chef Automate HTTP workers Chef server should allow to exist at any time.
     - Default: ``100``.
   * - ``data_collector['http_max_age']``
     - Maximum age a Chef Automate HTTP worker should be allowed to live, specified as an Erlang tuple.
     - Default: ``{70, sec}``.
   * - ``data_collector['http_cull_interval']``
     - How often Chef server should cull aged-out Chef Automate HTTP workers that have exceeded their ``http_max_age``, specified as an Erlang tuple.
     - Default: ``{1, min}``.
   * - ``data_collector['http_max_connection_duration']``
     - Maximum duration an HTTP connection is allowed to exist before it is terminated, specified as an Erlang tuple.
     - Default: ``{70, sec}``.


Next steps: :doc:`Perform a Compliance Scan </perform_compliance_scan>`

Troubleshooting: My data does not show up in the UI
=====================================================

.. tag chef_automate_visibility_no_data_troubleshoot

If an organization does not have any nodes associated with it, it does not show up in the **Nodes** section of the Chef Automate UI.
This is also true for roles, cookbooks, recipes, attributes, resources, node names, and environments. Only those items that have a node associated with them will appear in the UI. Chef Automate has all the data for all of these, but does not highlight them in the UI. This is designed to keep the UI focused on the nodes in your cluster.

.. end_tag

Next Steps
============================
:doc:`Data Collection with a Chef HA Cluster </data_collection_chef_ha`
:doc:`Data Collection without Chef Server </data_collection_without_server>`
:doc:`Data Collection with Habitat </data_collection_habitat>`

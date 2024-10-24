---
title: How to change {{ PG }} cluster settings in {{ mpg-full-name }}
description: In this tutorial, you will learn how to change settings for a {{ PG }} cluster.
---

# Updating {{ PG }} cluster settings

After creating a cluster, you can:

* [Change the host class](#change-resource-preset).

* [Configure servers](#change-postgresql-config) according to the [{{ PG }} documentation](https://www.postgresql.org/docs/current/runtime-config.html).

* [Change additional cluster settings](#change-additional-settings).

* [Manually switch the master host](#start-manual-failover).

* [Move a cluster](#move-cluster) to another folder.


* [Change cluster security groups](#change-sg-set).


Learn more about other cluster updates:

* [Upgrading the {{ PG }} version](cluster-version-update.md).

* [Managing disk space](storage-space.md).

* [Migrating cluster hosts to a different availability zone](host-migration.md).

## Changing the host class {#change-resource-preset}

{% note info %}

Some {{ PG }} settings [depend on the selected host class](../concepts/settings-list.md#settings-instance-dependent).

{% endnote %}

When changing the host class:

* Your single-host cluster will be unavailable for a few minutes with database connections terminated.
* Your multi-host cluster will get a new master host. Its hosts will be stopped and updated one by one. Once stopped, a host will be unavailable for a few minutes.
* Using a [special FQDN](./connect.md#special-fqdns) does not guarantee a stable database connection: user sessions may be terminated.

We recommend changing the host class only when the cluster has no active workload.

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Select the cluster and click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.mdb.cluster.overview.button_action-edit }}** in the top panel.
   1. Under **{{ ui-key.yacloud.mdb.forms.section_resource }}**, select the required class for the {{ PG }} hosts.
   1. Click **{{ ui-key.yacloud.mdb.forms.button_edit }}**.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To change the [host class](../concepts/instance-types.md) for the cluster:

   1. View a description of the update cluster CLI command:

      ```bash
      {{ yc-mdb-pg }} cluster update --help
      ```

   1. Request a list of available host classes (the `ZONE IDS` column specifies the availability zones where you can select the appropriate class):


      ```bash
      {{ yc-mdb-pg }} resource-preset list
      ```

      ```text
      +-----------+--------------------------------+-------+----------+
      |    ID     |            ZONE IDS            | CORES |  MEMORY  |
      +-----------+--------------------------------+-------+----------+
      | s1.micro  | {{ region-id }}-a, {{ region-id }}-b,  |     2 | 8.0 GB   |
      |           | {{ region-id }}-d                  |       |          |
      | ...                                                           |
      +-----------+--------------------------------+-------+----------+
      ```


   1. Specify the class in the update cluster command:

      ```bash
      {{ yc-mdb-pg }} cluster update <cluster_name_or_ID> \
          --resource-preset <host_class_ID>
      ```

      {{ mpg-short-name }} will run the update host class command for the cluster.

- {{ TF }} {#tf}

   1. Open the current {{ TF }} configuration file with an infrastructure plan.

      For more information about how to create this file, see [Creating clusters](cluster-create.md).

      For a complete list of available {{ mpg-name }} cluster configuration fields, see the [{{ TF }} provider documentation]({{ tf-provider-mpg }}).

   1. In the {{ mpg-name }} cluster description, change the `resource_preset_id` attribute value under `config.resources`:

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        config {
          resources {
            resource_preset_id = "<host_class>"
            ...
          }
        }
      }
      ```

   1. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

   To change the class of cluster hosts, use the [update](../api-ref/Cluster/update.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Update](../api-ref/grpc/Cluster/update.md) gRPC API call and provide the following in the request:

   * Cluster ID in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](./cluster-list.md#list-clusters).
   * Host class ID in the `configSpec.resources.resourcePresetId` parameter. To request a list of supported values, use the [list](../api-ref/ResourcePreset/list.md) method for the `ResourcePreset` resources.
   * List of settings to update (in this case, `configSpec.resources.resourcePresetId`) in the `updateMask` parameter.

   {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}

## Changing {{ PG }} settings {#change-postgresql-config}

You can change the DBMS settings of the hosts in your cluster.

{% note warning %}

* You cannot change {{ PG }} settings using SQL commands.
* Some {{ PG }} settings [depend on the selected host class or storage size](../concepts/settings-list.md#settings-instance-dependent).

{% endnote %}

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Select the cluster and click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.mdb.cluster.overview.button_action-edit }}** in the top panel.
   1. Change the [{{ PG }} settings](../concepts/settings-list.md) by clicking **{{ ui-key.yacloud.mdb.forms.button_configure-settings }}** under **{{ ui-key.yacloud.mdb.forms.section_settings }}**.
   1. Click **{{ ui-key.yacloud.component.mdb.settings.popup_settings-submit }}**.
   1. Click **{{ ui-key.yacloud.mdb.forms.button_edit }}**.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To update the [{{ PG }} settings](../concepts/settings-list.md):

   1. View the full list of settings specified for the cluster:

      ```bash
      {{ yc-mdb-pg }} cluster get <cluster_name_or_ID> --full
      ```

   1. View a description of the update cluster configuration CLI command:

      ```bash
      {{ yc-mdb-pg }} cluster update-config --help
      ```

   1. Set the required parameter values:

      All supported parameters are listed in the [request format for the update method](../api-ref/Cluster/update.md), in the `postgresqlConfig_<{{ PG }}_version>` field. To specify a parameter name in the CLI call, convert the name from <q>lowerCamelCase</q> to <q>snake_case</q>. For example, the `maxPreparedTransactions` parameter from an API call should be converted to `max_prepared_transactions` for the CLI command:

      ```bash
      {{ yc-mdb-pg }} cluster update-config <cluster_name_or_ID> \
         --set <name_of_parameter_1>=<value_1>,<name_of_parameter_2>=<value_2>,...
      ```

      {{ mpg-short-name }} runs the update cluster settings operation.

- {{ TF }} {#tf}

   1. Open the current {{ TF }} configuration file with an infrastructure plan.

      For more information about how to create this file, see [Creating clusters](cluster-create.md).

      For a complete list of available {{ mpg-name }} cluster configuration fields, see the [{{ TF }} provider documentation]({{ tf-provider-mpg }}).

   1. In the {{ mpg-short-name }} cluster description, change the values of the parameters under `config.postgresql_config`. If there is no such section, create one.

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        config {
          ...
          postgresql_config = {
            max_connections                   = <maximum_number_of_connections>
            enable_parallel_hash              = <true_or_false>
            vacuum_cleanup_index_scale_factor = <number_from_0_to_1>
            ...
          }
        }
      }
      ```

   1. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

   To change {{ PG }} server settings, use the [update](../api-ref/Cluster/update.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Update](../api-ref/grpc/Cluster/update.md) gRPC API call and provide the following in the request:

   * Cluster ID in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](./cluster-list.md#list-clusters).
   * Required setting values in the `configSpec.postgresqlConfig_<{{ PG }}_version>` parameter.
   * List of settings to update, in the `updateMask` parameter.

   {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}

## Changing additional cluster settings {#change-additional-settings}

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Select the cluster and click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.mdb.cluster.overview.button_action-edit }}** in the top panel.
   1. Change additional cluster settings:

      {% include [mpg-extra-settings](../../_includes/mdb/mpg/extra-settings-web-console.md) %}

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To change additional cluster settings:

   1. View a description of the update cluster CLI command:

      ```bash
      {{ yc-mdb-pg }} cluster update --help
      ```

   1. Run the following command with a list of settings to update:


      ```bash
      {{ yc-mdb-pg }} cluster update <cluster_name_or_ID> \
          --backup-window-start <backup_start_time> \
          --datalens-access=<true_or_false> \
          --maintenance-window type=<maintenance_type>,`
                              `day=<day_of_week>,`
                              `hour=<hour> \
          --websql-access=<true_or_false> \
          --deletion-protection=<deletion_protection> \
          --connection-pooling-mode=<connection_pooler_mode> \
          --serverless-access=<true_or_false> \
          --yandexquery-access=<access_through_{{ yq-name }}> \
          --performance-diagnostics enabled=<true_or_false>,`
                                   `sessions-sampling-interval=<session_sampling_interval>,`
                                   `statements-sampling-interval=<statement_sampling_interval>
      ```



   You can change the following settings:

   {% include [backup-window-start](../../_includes/mdb/cli/backup-window-start.md) %}

   * `--datalens-access`: Enables DataLens access. The default value is `false`. For more information on setting up a connection, see [Connecting to a cluster from {{ datalens-name }}](datalens-connect.md).

   * `--maintenance-window`: [Maintenance window](../concepts/maintenance.md) settings (including for disabled clusters), where `type` is the maintenance type:

      {% include [maintenance-window](../../_includes/mdb/cli/maintenance-window-description.md) %}

   * `--websql-access`: Enables you to [run SQL queries](web-sql-query.md) to cluster databases from the {{ yandex-cloud }} management console using {{ websql-full-name }}. The default value is `false`.


   * `--serverless-access`: Enables cluster access from [{{ sf-full-name }}](../../functions/concepts/index.md). The default value is `false`. For more information about setting up access, see the [{{ sf-name }}](../../functions/operations/database-connection.md) documentation.

   * `--yandexquery-access`: Enables cluster access from [{{ yq-full-name }}](../../query/concepts/index.md). This feature is at the [Preview](../../overview/concepts/launch-stages.md) stage and available upon request.


   * `--autofailover` manages automatic master change setup. To learn more, see [Replication](../concepts/replication.md#replication-auto). The default value is `true`.

   * `--connection-pooling-mode`: Specifies the [connection pooler mode](../concepts/pooling.md) (`SESSION`, `TRANSACTION`, or `STATEMENT`).

   * {% include [Deletion protection](../../_includes/mdb/cli/deletion-protection.md) %}

      By default, the parameter inherits its value from the cluster when creating users and databases. You can also set the value manually; for more information, see the [User management](cluster-users.md) and [Database management](databases.md) sections.

      If the parameter is changed on a running cluster, only users and databases with the **Same as cluster** protection will inherit the new value.

      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-db.md) %}

   * `--performance-diagnostics`: [Statistics collection](./performance-diagnostics.md#activate-stats-collector) settings:

      * `enabled`: `true` value enables statistics collection. The default value is `false`.
      * `sessions-sampling-interval`: Session sampling interval, seconds. Acceptable values are between `1` and `86400`.
      * `statements-sampling-interval`: Statement sampling interval, seconds. Acceptable values are between `60` and `86400`.

   You can [retrieve the cluster name with a list of clusters in the folder](cluster-list.md#list-clusters).

- {{ TF }} {#tf}

   1. Open the current {{ TF }} configuration file with an infrastructure plan.

      For more information about how to create this file, see [Creating clusters](cluster-create.md).

      For a complete list of available {{ mpg-name }} cluster configuration fields, see the [{{ TF }} provider documentation]({{ tf-provider-mpg }}).

   1. To change the backup start time, add a block named `config.backup_window_start` to the {{ mpg-name }} cluster description.

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        config {
          backup_window_start {
            hours   = <hour_of_backup_start_time>
            minutes = <minute_of_backup_start_time>
          }
          ...
        }
      }
      ```

   1. To enable access from {{ datalens-full-name }} and allow [execution of SQL queries from the management console](web-sql-query.md) using {{ websql-full-name }}, change the values of the appropriate fields in the `config.access` section:

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        config {
          access {
            data_lens = <access_from_{{ datalens-name }}>
            web_sql   = <execution_of_SQL_queries_from_management_console>
            ...
        }
        ...
      }
      ```

      Where:

      * `data_lens`: Access from {{ datalens-name }}, `true` or `false`.
      * `web_sql`: Execution of SQL queries from the management console using {{ websql-full-name }}: `true` or `false`.

   1. To change the [connection pooler mode](../concepts/pooling.md), add the `config.pooler_config` section to the {{ mpg-name }} cluster description:

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        config {
          pooler_config {
            pool_discard = <Odyssey_parameter>
            pooling_mode = "<pooling_mode>"
          }
          ...
        }
      }
      ```

      Where:

      * `pool_discard`: Odyssey `pool_discard` parameter, `true` or `false`.
      * `pooling_mode`: Pooling mode: `SESSION`, `TRANSACTION`, or `STATEMENT`.

   1. {% include [Maintenance window](../../_includes/mdb/mpg/terraform/maintenance-window.md) %}

   1. {% include [Performance diagnostics](../../_includes/mdb/mpg/terraform/performance-diagnostics.md) %}

   1. To enable protection of the cluster, its databases, and users against accidental deletion, add the `deletion_protection` field set to `true` to your cluster description:

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        deletion_protection = <deletion_protection>
      }
      ```

      Where `deletion_protection` is the protection of the cluster, its databases, and users against deletion. It may take either the `true` or `false` value.

      By default, the parameter inherits its value from the cluster when creating users and databases. You can also set the value manually; for more information, see the [User management](cluster-users.md) and [Database management](databases.md) sections.

      If the parameter is changed on a running cluster, only users and databases with the **Same as cluster** protection will inherit the new value.

      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-db.md) %}

   1. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

   To change additional cluster settings, use the [update](../api-ref/Cluster/update.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Update](../api-ref/grpc/Cluster/update.md) gRPC API call and provide the following in the request:

   * Cluster ID in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](./cluster-list.md#list-clusters).
   * Settings for access from other services in the `configSpec.access` parameter.
   * Backup window settings in the `configSpec.backupWindowStart` parameter.
   * [Connection pooler mode](../concepts/pooling.md) in the `configSpec.poolerConfig.poolingMode` parameter.
   * [Maintenance](../concepts/maintenance.md) window settings (including for disabled clusters) in the `maintenanceWindow` parameter.
   * Settings for protection of the cluster, its databases, and users against deletion in the `deletionProtection` parameter: `true` or `false`.

      By default, the parameter inherits its value from the cluster when creating users and databases. You can also set the value manually; for more information, see the [User management](cluster-users.md) and [Database management](databases.md) sections.

      If the parameter is changed on a running cluster, only users and databases with the **Same as cluster** protection will inherit the new value.

      {% include [deletion-protection-limits-db](../../_includes/mdb/deletion-protection-limits-db.md) %}

   * List of cluster configuration fields to update in the `UpdateMask` parameter.

   {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}


   To allow cluster access from [{{ sf-full-name }}](../../functions/concepts/index.md), set `true` for the `configSpec.access.serverless` parameter. For more information about setting up access, see the [{{ sf-name }}](../../functions/operations/database-connection.md) documentation.

   To allow cluster access from [{{ yq-full-name }}](../../query/index.yaml), set `true` for the `configSpec.access.yandexQuery` parameter. This feature is at the [Preview](../../overview/concepts/launch-stages.md) stage and available upon request.


   To enable [statistics collection](./performance-diagnostics.md#activate-stats-collector):

   {% include [Performance diagnostic API](../../_includes/mdb/mpg/performance-diagnostics-api.md) %}


{% endlist %}

## Manually switching the master {#start-manual-failover}

In a fault-tolerant {{ mpg-name }} cluster with multiple hosts, you can switch the master role from the current master host to one of the replicas. After this operation, the current master host becomes the replica host of the new master.

Specifics of switching master hosts in {{ mpg-name }}

* You cannot switch the master host to a replica for which the source of the replication stream is explicitly given.
* If you do not specify the replica host name explicitly, the master host will switch to one of the quorum replicas.

To learn more, see [Replication](../concepts/replication.md).

To switch the master:

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Click the cluster name and select the ![icon-hosts.svg](../../_assets/console-icons/cube.svg) **{{ ui-key.yacloud.postgresql.cluster.switch_hosts }}** tab.
   1. Click ![icon-autofailover.svg](../../_assets/console-icons/shuffle.svg) **{{ ui-key.yacloud.mdb.cluster.hosts.button_manual-failover }}**.
      * To switch the master to one of the quorum replicas, leave the **{{ ui-key.yacloud.mdb.dialogs.popup-confirm-switch-master_auto }}** option enabled.
      * To switch the master to a specific replica, disable the **{{ ui-key.yacloud.mdb.dialogs.popup-confirm-switch-master_auto }}** option and then select the required replica from the drop-down list.
   1. Click **{{ ui-key.yacloud.mdb.dialogs.popup-confirm-switch-master_button }}**.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   Run this command:

   ```bash
   {{ yc-mdb-pg }} cluster start-failover <cluster_name_or_ID> \
       --host <replica_host_name>
   ```

   You can request the replica host name with a [list of cluster hosts](hosts.md#list) and the cluster name with a [list of clusters in the folder](cluster-list.md#list-clusters).

- {{ TF }} {#tf}

   1. Open the current {{ TF }} configuration file with an infrastructure plan.

      For more information about how to create this file, see [Creating clusters](cluster-create.md).

      For a complete list of available {{ mpg-name }} cluster configuration fields, see the [{{ TF }} provider documentation]({{ tf-provider-mpg }}).

   1. In the `host_master_name` parameter, specify the name of the replica host to switch to.

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        host_master_name = "<replica_host_name>"
      }
      ```

      Where `host_master_name` is the name of the replica host, i.e., the `name` attribute of the appropriate `host` block.

   1. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

   To switch the master host, use the [startFailover](../api-ref/Cluster/startFailover.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/StartFailover](../api-ref/grpc/Cluster/startFailover.md) gRPC API call and provide the following in the request:

   * ID of the cluster where you want to switch the master, in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](cluster-list.md#list-clusters).
   * (Optional) In the `hostName` parameter, the name of the replica host to switch to. To find out the name, [get a list of hosts in the cluster](hosts.md#list).

{% endlist %}

## Moving a cluster {#move-cluster}

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Click ![image](../../_assets/console-icons/ellipsis.svg) to the right of the cluster you want to move.
   1. Select **{{ ui-key.yacloud.mdb.clusters.button_action-move }}**.
   1. Select a folder you want to move the cluster to.
   1. Click **{{ ui-key.yacloud.mdb.dialogs.popup_button_move-cluster }}**.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To move a cluster:

   1. View a description of the CLI move cluster command:

      ```bash
      {{ yc-mdb-pg }} cluster move --help
      ```

   1. Specify the destination folder in the move cluster command:

      ```bash
      {{ yc-mdb-pg }} cluster move <cluster_ID> \
         --destination-folder-name=<destination_folder_name>
      ```

      You can get the cluster ID with a [list of clusters in the folder](cluster-list.md#list-clusters).

- API {#api}

   To move a cluster, use the [move](../api-ref/Cluster/move.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Move](../api-ref/grpc/Cluster/move.md) gRPC API call and provide the following in the request:

   * Cluster ID in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](cluster-list.md#list-clusters).
   * ID of the destination folder in the `destinationFolderId` parameter.

{% endlist %}

After the cluster is moved, it will continue using the cloud network from the source folder. If you want to host the cluster in a different cloud network, use the [restore from a backup](./cluster-backups.md) feature and specify the required network for the cluster backup.

To move a cluster to a different availability zone, follow [this guide](host-migration.md). You will thus move the cluster hosts.


## Changing security groups {#change-sg-set}

{% list tabs group=instructions %}

- Management console {#console}

   1. Go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-postgresql }}**.
   1. Select the cluster and click ![image](../../_assets/console-icons/pencil.svg) **{{ ui-key.yacloud.mdb.cluster.overview.button_action-edit }}** in the top panel.
   1. Under **{{ ui-key.yacloud.mdb.forms.section_network }}**, select security groups for cluster network traffic.

- CLI {#cli}

   {% include [cli-install](../../_includes/cli-install.md) %}

   {% include [default-catalogue](../../_includes/default-catalogue.md) %}

   To edit the list of [security groups](../concepts/network.md#security-groups) for your cluster:

   1. View a description of the update cluster CLI command:

      ```bash
      {{ yc-mdb-pg }} cluster update --help
      ```

   1. Specify the security groups in the update cluster command:

      ```bash
      {{ yc-mdb-pg }} cluster update <cluster_name_or_ID> \
          --security-group-ids <list_of_security_group_IDs>
      ```

- {{ TF }} {#tf}

   1. Open the current {{ TF }} configuration file with an infrastructure plan.

      For more information about how to create this file, see [Creating clusters](cluster-create.md).

      For a complete list of available {{ mpg-name }} cluster configuration fields, see the [{{ TF }} provider documentation]({{ tf-provider-mpg }}).

   1. Change the value of the `security_group_ids` parameter in the cluster description:

      ```hcl
      resource "yandex_mdb_postgresql_cluster" "<cluster_name>" {
        ...
        security_group_ids = [ <list_of_security_group_IDs> ]
      }
      ```

   1. Make sure the settings are correct.

      {% include [terraform-validate](../../_includes/mdb/terraform/validate.md) %}

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [Terraform timeouts](../../_includes/mdb/mpg/terraform/timeouts.md) %}

- API {#api}

   To edit the list of cluster security groups, use the [update](../api-ref/Cluster/update.md) REST API method for the [Cluster](../api-ref/Cluster/index.md) resource or the [ClusterService/Update](../api-ref/grpc/Cluster/update.md) gRPC API call and provide the following in the request:

   * Cluster ID in the `clusterId` parameter. To find out the cluster ID, [get a list of clusters in the folder](cluster-list.md#list-clusters).
   * List of security group IDs in the `securityGroupIds` parameter.
   * List of settings to update (in this case, `securityGroupIds`) in the `updateMask` parameter.

   {% include [Note API updateMask](../../_includes/note-api-updatemask.md) %}

{% endlist %}

{% note warning %}

You may need to additionally [set up security groups](connect.md#configuring-security-groups) to connect to the cluster.

{% endnote %}


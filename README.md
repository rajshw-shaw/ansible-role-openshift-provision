openshift-provision
=========

An Ansible role for provisioning resources within OpenShift clusters.

Installation
------------

```
ansible-galaxy install https://github.com/jkupferer/ansible-role-openshift-provision/archive/master.tar.gz#/openshift-provision
```

Requirements
------------

OCP 3.3+
ansible 2.4+

Role Variables
--------------

* `resource_definition` - Ansible variables file definining resources to
  create by use of `include_vars`

* `openshift_clusters` - List of openshift cluster definitions as defined
  below

* `openshift_connection_certificate_authority` - Path to file containing
  signing certificate for OpenShift server. This option may also be set within
  `openshift_clusters` as `connection.certificate_authority`

* `openshift_connection_insecure_skip_tls_verify` - If set to "true" then
  disable SSL/TLS checks when connectiong to OpenShift server. This option
  may also be set within `openshift_clusters` as
  `connection.insecure_skip_tls_verify`

* `openshift_connection_server` - Server URL for connecting to the OpenShift
  cluster. Should by of the form "https://master.example.com:8443". This option
  may also be set within `openshift_clusters` as `connection.server`

* `openshift_connection_token` - OpenShift user token. This option may also be
  set within `openshift_clusters` as `connection.token`

* `openshift_login_username` - Login username. Use of a connection token is
  preferred to providing a username and password. This option may also be
  set within `openshift_clusters` as `login.username`

* `openshift_login_password` - Login password. Use of a connection token is
  preferred to providing a username and password. This option may also be
  set within `openshift_clusters` as `login.password`

* `user_groups` (DEPRECATED) - List of groups to create across all clusters,
  use of `groups` under `openshift_clusters` is preferred 

### `openshift_clusters[*]`

List of OpenShift cluster definitions

* `connection` - OpenShift connection parameters, defined to match `oc` command
  line including `server`, `certificate_authority`, `insecure_skip_tls_verify`,
  and `token`. If omitted then it is assumed the environment is already has a
  valid OpenShift command line login session

* `login` - Credentials for login to the OpenShift cluster, `username` and
  `password`

* `openshift_host_env` - String or array of strings specifying hostnames. The
  cluster resources will only be provisioned if this variable is not set or if
  `openshift_master_cluster_public_hostname` equals a value in
  `openshift_host_env`

* `cluster_resources` - List of OpenShift resource definitions, these should
  be cluster level resources

* `cluster_role_bindings` - List of roles and assigned users and groups,
  described below

* `groups` - List of OpenShift groups to create along with group membership,
  described below

* `projects` - List of projects to manage, described below:

* `resources` - List of OpenShift resource definitions, these should be
  project level resources that define the namespace. Normally project resources
  should appear within `projects`, but sometimes specific ordering of resource
  creation may be desired

* `persistent_volumes` (DEPRECATED) - List of persistent volumes. Use of
  `cluster_resources` is preferred

* `cluster_resource_quotas` (DEPRECATED) - List of cluster resource quota
  definitions. Use of `cluster_resources` is preferred

* `cluster_roles` (DEPRECATED) - List of OpenShift cluster role definitions.
  Use of `cluster_resources` is preferred

### `openshift_clusters[*].cluster_resources`

Cluster resources are the first items processed in provisioning. This is a
list of OpenShift resource definitions that are created/updated using the `oc`
command. The default action is `oc apply`, but may be overridden by setting
`action` on the resource to `create` or `replace`. If `action` is set to
`create` and the cluster resource already exists then no action is taken.
Besides the field `action` all other fields follow OpenShift standards. All
resources must define `metadata.name`.

### `openshift_clusters[*].cluster_role_bindings`

List of cluster role assignments. Each entry is a dictionary containing:

* `role` - Name of role managed. This is *not* the name name of rolebinding,
  but rather the name of the role referenced by the role binding. Assignment
  and removal of roles uses `oc adm policy` commands which do not specify the
  name of role bindings. Required

* `users` - List of user names that should be granted access to this cluster
  role. Optional

* `groups` - List of group names that should be granted access to this cluster
  role. Optional

* `remove_unlisted` - Boolean to indicate whether users or groups not listed
  should be removed from any current access to this cluster role. Optional,
  default "false"

* `remove_unlisted_users` - Same as `remove_unlisted`, but specifically
  targeting users. Optional, default "false"

* `remove_unlisted_groups` - Same as `remove_unlisted`, but specifically
  targeting groups. Optional, default "false"

### `openshift_clusters[*].groups`

List of OpenShift groups to manage

* `name` - Group name. Required

* `members` - List of user names that should belong to this group. Optional

* `remove_unlisted_members` - Boolean to indicate whether unlisted users should
  be removed from this group. Optional, default "false"

### `openshift_clusters[*].projects`

* `name` - Project name string

* `annotations` - Dictionary of project annotations

* `description` - Project description string

* `display_name` - Project display name string

* `join_pod_network` - Name of target project to which this project network
  should be joined for use with multi-tenant SDN

* `labels` - Dictionary of project labels

* `node_selector` - Node selector to apply to the project namespace

* `process_templates` - Templates to process to create resources within this
  project, described below

* `resources` - Definitions of OpenShift resources to create in project,
  described below

* `role_bindings` - Role bindings to apply to project to grant or revoke user
  and group access to roles, described below

* `service_accounts` - List of service accounts to provision within this
  project. Each entry is a name of a service account to create. Service
  accounts may also be created with `resources`

* `limit_ranges` (DEPRECATED) - List of limit ranges to apply to project, use
  of `resources` is preferred to create LimitRange objects

* `persistent_volume_claims` (DEPRECATED) - List of persistent volume claims
  for project, use of `resources` is preferred to create PersistentVolumeClaim
  objects

* `quotas` (DEPRECATED) - List of quotas to apply to project, use of
  `resources` is preferred to create ResourceQuota objects

### `openshift_clusters[*].projects[*].process_templates`

List of templates to process to manage resources within project. Resources
within the project are managed with `oc apply` command by default. An alternate
action may be specified with the `action` parameter on the `process_templates`
entry, which may be set to `create` or `replace`. If `create` is specified then
the the template will only be processed if no resources created by the template
are found. Template resources are determined by label, `template` which should
be set to the template name, so all templates processed in this way should
define a `template` label on resources the template creates.

* `name` - Template name to process

* `namespace` - Namespace in which the template is found, default is this
  project

* `parameters` - Dictionary of parameters to pass to the template. Optional

* `action` - Action to process template output, values may be "create",
  "apply", or "replace". Default "apply"

* `cascade` - Boolean flag to activate `--cascade` option to `oc apply` or
  `oc replace`

* `force` - Boolean flag to activate `--force` option to `oc apply` or
  `oc replace`

* `overwrite` - Boolean flag to activate `--overwrite` option to `oc apply`

* `prune` - Boolean flag to activate `--prune` option to `oc apply`

* `prune_whitelist` - List of arguments to pass to `--prune-whitelist` for use with `oc apply`

### `openshift_clusters[*].projects[*].resources`

This is a list of OpenShift resource definitions that are created/updated in
a project using the `oc` command. The default action is `oc apply`, but may be
overridden by setting `action` on the resource to `create` or `replace`. If
`action` is set to `create` and the project resource already exists then no
action is taken. Besides the field `action` all other fields follow OpenShift
standards. All resources must define `metadata.name`.

### `openshift_clusters[*].projects[*].role_bindings`

List of project role assignments. Each entry is a dictionary containing:

* `role` - Name of role managed. This is *not* the name name of rolebinding,
  but rather the name of the role referenced by the role binding. Assignment
  and removal of roles uses `oc policy` commands which do not specify the
  name of role bindings. Required

* `users` - List of user names that should be granted access to this project
  role. Optional

* `groups` - List of group names that should be granted access to this project
  role. Optional

* `remove_unlisted` - Boolean to indicate whether users or groups not listed
  should be removed from any current access to this project role. Optional,
  default "false"

* `remove_unlisted_users` - Same as `remove_unlisted`, but specifically
  targeting users. Optional, default "false"

* `remove_unlisted_groups` - Same as `remove_unlisted`, but specifically
  targeting groups. Optional, default "false"

### `openshift_clusters[*].resources`

List of OpenShift project resources to create. Declaration is the same as
specified above for `openshift_clusters[*].projects[*].resources` with the
addition that each entry here must specify `metadata.namespace` to specify
the target project for the resource.

Example Playbook
----------------

    - hosts: masters[0]
      roles:
         - role: openshift-logging-elasticsearch-hostmount
           resource_definition: ocp-resouces/app.yml

Example resources file:

    openshift_clusters:
    - connection:
        server: https://openshift-master.libvirt:8443
        token: abcdefghijklmnopqrstuvwxyz0123456798...

      cluster_resources:
      - apiVersion: v1
        kind: ClusterRole
        metadata:
          creationTimestamp: null
          name: network-joiner
        rules:
        - apiGroups:
          - network.openshift.io
          - ""
          attributeRestrictions: null
          resources:
          - netnamespaces
          verbs:
          - create
          - delete
          - get
          - list
          - update
        - apiGroups:
          - ""
          attributeRestrictions: null
          resources:
          - namespaces
          - projects
          verbs:
          - get
          - list
        - apiGroups:
          - network.openshift.io
          - ""
          attributeRestrictions: null
          resources:
          - clusternetworks
          verbs:
          - get
      - apiVersion: v1
        kind: ClusterResourceQuota
        metadata:
          creationTimestamp: null
          name: serviceaccount-app-jenkins
        spec:
          quota:
            hard:
              limits.cpu: "10"
              limits.memory: 20Gi
              requests.cpu: "5"
              requests.memory: 20Gi
          selector:
            annotations:
              openshift.io/requester: system:serviceaccount:app-dev:jenkins
            labels: null
      - apiVersion: v1
        kind: PersistentVolume
        metadata:
          creationTimestamp: null
          labels:
            foo: bar
          name: nfs-foo
        spec:
          access_modes:
          - ReadWriteMany
          capacity:
            storage: 10Gi
          nfs:
            path: /export/foo
            server: nfsserver.example.com
          persistentVolumeReclaimPolicy: Retain

      cluster_role_bindings:
      - role: self-provisioner
        users:
        - system:serviceaccount:app-dev:jenkins
      - role: network-joiner
        users:
        - system:serviceaccount:app-dev:jenkins
        groups:
        - app-admin
        remove_unlisted: true

      groups:
      - name: app-admin
        remove_unlisted_members: true
        members:
        - alice
        - bob

      projects:
      - name: app-dev
        description: Application Description
        display_name: Application Name
        labels:
          application: appname
        node_selector: region=app

        process_templates:
        - name: httpd-example
          namespace: openshift
          parameters:
            SOURCE_REPOSITORY_URL: https://github.com/openshift/httpd-ex.git

        resources:
        - appVersion: v1
          kind: ResourceQuota
          metadata:
            name: compute
          spec:
            hard:
              requests.cpu: "10"
              requests.memory: "50Gi"
              limits.cpu: "20"
              limits.memory: "50Gi"
        - appVersion: v1
          kind: LimitRange
          metadata:
            name: compute
          spec:
            limits:
            - type: Pod
              min:
                cpu: 50m
                memory: 4Mi
              max:
                cpu: "2"
                memory: 5Gi
            - type: Container
              min:
                cpu: 50m
                memory: 4Mi
              max:
                cpu: "2"
                memory: 5Gi
              default:
                cpu: "1"
                memory: 1Gi
              defaultRequest:
                cpu: 200m
                memory: 1Gi

        role_bindings:
        - role: admin
          groups: app-admin
          remove_unlisted: true
        - role: edit
          users:
          - system:serviceaccount:app-dev:jenkins
          remove_unlisted_users: true
        - role: view
          groups:
          - app-developer
    
        service_accounts:
        - jenkins

License
-------

BSD

Author Information
------------------

Johnathan Kupferer (jkupfere@redhat.com)

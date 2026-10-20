# What level of permissions should users have — admin or user?

**Applies to:** Red Hat OpenShift AI 3.5 Self-Managed.

Permissions in OpenShift AI should follow the principle of least privilege. Most users — data scientists and MLOps engineers — should have **User** permissions. **Administrator** should go only to the people responsible for platform configuration, and **`cluster-admin`** only to your OpenShift platform team.

The platform separates platform-wide access from project-specific permissions.

## Platform-wide user types

| User type | Recommended for | Key permissions |
|---|---|---|
| **Users** | Data scientists, MLOps engineers | Access and use individual components: workbenches, AI pipelines, model deployment, the model catalog, the Gen AI playground. |
| **Administrators** | Lead data scientists, AI platform owners | Everything a user can do, plus configure OpenShift AI settings, access and manage **any user's** workbench, and manage pipeline applications for **any** project. |
| **Cluster admins** | IT / OpenShift administrators | Install and upgrade the operator, enable components in the DataScienceCluster, enable GPU accelerators, add and remove users from groups, and manage cluster-level resources such as PVCs and ConfigMaps. |

Two default behaviours are worth designing around:

- **By default, every OpenShift-authenticated user can access OpenShift AI** — the user group ships as `system:authenticated`, and the admin group as `rhods-admins`. To gate who can use the platform, select a named OpenShift group instead. See [Selecting admin and user groups](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/selecting-admin-and-user-groups_resource-mgmt) and [Adding users to user groups](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_openshift_ai/managing-users-and-groups#adding-users-to-user-groups_managing-rhoai).
- **Anyone with `cluster-admin` is automatically an OpenShift AI administrator.** This is the main reason not to give `cluster-admin` to end users — it silently grants full platform administration as well.

OpenShift AI only selects groups that already exist in OpenShift, so groups backed by your identity provider are set up at the OpenShift level rather than in OpenShift AI — see [Understanding identity provider configuration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/authentication_and_authorization/understanding-identity-provider), and [Syncing LDAP groups](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/authentication_and_authorization/ldap-syncing) if you use LDAP. If you manage the cluster declaratively, both group settings live in the cluster-scoped `Auth` resource (`services.platform.opendatahub.io/v1alpha1`), which is what *Settings → User management* writes to.

## Project-level permissions

Each project is authorised separately on its **Permissions** tab, for either a user or a group:

| Role | Grants |
|---|---|
| **Admin** | Edit project details and manage access to the project. |
| **Contributor** | View and edit project components: workbenches, connections, storage. |

There's no built-in read-only project role, and changing project permissions doesn't require `cluster-admin` — the project owner or an OpenShift AI administrator can do it.

If you need finer granularity than this, 3.5 adds [custom roles](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_on_projects/managing-access-to-projects_projects#assigning-custom-roles-to-project-users_projects) that you can create from the dashboard, including read-only and update-only workbench templates.

## What actually needs cluster-admin

This matters for how you staff day-2 operations, and a few of these are not where people expect them.

**Needs `cluster-admin`:** installing, uninstalling and upgrading the operator; editing the DataScienceCluster; adding and removing users from OpenShift groups; enabling GPU accelerators; managing Kueue quota; creating role bindings to model registry roles.

**An OpenShift AI administrator can do without it:** selecting which groups are the admin and user groups; **creating and managing hardware profiles**; **adding custom model-serving runtimes**; creating and editing model registries; dashboard customisation, connection types and storage classes; starting, stopping and accessing other users' workbenches.

## Read more

- [Platform user types and permissions](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_openshift_ai/managing-users-and-groups#overview-of-user-types-and-permissions_managing-rhoai)
- [Selecting admin and user groups](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/selecting-admin-and-user-groups_resource-mgmt)
- [Managing access to projects](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_on_projects/managing-access-to-projects_projects#granting-access-to-a-project_projects) — Admin, Contributor and custom roles

# ansible-role-infosvr-openigc

Ansible role for automating the deployment of OpenIGC objects for the Information Governance Catalog, within IBM Information Server.

## Requirements

- Ansible v2.4.x
- Root-become-able network access to an IBM Information Server environment

(To ensure installation of required utilities like zip, rsync and curl)

## Role Variables

See `defaults/main.yml` for inline documentation, and the example below for the main variables needed.

## Example Playbook

The role is primarily inteded to be imported into other playbooks as-needed for the deployment of OpenIGC objects -- both bundles and asset instances. (Thus the need for Ansible v2.4.x and the `import_role` module.)

```
- import_role: name=cmgrote.ibm-infosvr-openigc
  vars:
    ibm_infosvr_openigc_services_host: myhost.domain.com
    ibm_infosvr_openigc_services_console_port: 9445
    ibm_infosvr_openigc_admin_user: isadmin
    ibm_infosvr_openigc_admin_user_pwd: "{{ some_pwd_from_vault }}"
    ibm_infosvr_openigc_dsadm_user: dsadm
    ibm_infosvr_openigc_dsadm_group: dstage
    ibm_infosvr_openigc_bundle_directories:
      - "/some/directory/<BundleId>"
    ibm_infosvr_openigc_asset_instances:
      - "/some/directory/asset_instances-<BundleId>.xml"
```

## License

Apache 2.0

## Author Information

Christopher Grote

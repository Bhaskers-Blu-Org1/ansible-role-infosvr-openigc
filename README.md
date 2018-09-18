# ansible-role-infosvr-openigc

Ansible role for automating the deployment of OpenIGC objects for the Information Governance Catalog, within IBM Information Server.

## Requirements

- Ansible v2.4.x
- The following utilities pre-installed on your control machine:
  - zip
  - curl (until Ansible v2.7+ where uri natively supports binary file uploads)

The role does not use any privilege escalation, and executes primarily on the control machine itself (only pushing generate files directly against the relevant APIs in the environment).

## Role Variables

See `defaults/main.yml` for inline documentation, and the example below for the main variables needed.

This role will attempt to automatically re-use variables from the [IBM.infosvr](https://galaxy.ansible.com/IBM/infosvr) role, for instance by using a playbook that first imports `IBM.infosvr` using `tasks_from=setup_vars.yml`.

## Example Playbook

The role is primarily inteded to be imported into other playbooks as-needed for the deployment of OpenIGC objects -- both bundles and asset instances. (Thus the need for Ansible v2.4.x and the `import_role` module.)

```yml
---
- name: setup Information Server vars
  hosts: all
  tasks:
    - import_role: name=IBM.infosvr tasks_from=setup_vars.yml

- name: load OpenIGC bundles and assets
  hosts: ibm-information-server-engine
  roles:
    - IBM.infosvr-openigc
  vars:
    ibm_infosvr_openigc_bundle_directories:
      - /some/directory/<BundleId>
    ibm_infosvr_openigc_asset_instances:
      - /some/directory/asset_instances-<BundleId>.xml
    ibm_infosvr_openigc_assets_as_yaml:
      - /some/directory/<BundleId>.yml
```

### `ibm_infosvr_openigc_bundle_directories`

The list of directories provided through this variable should be in bundle form, specifically containing the following structure:

- `<BundleId>`: the root directory you point to should have the same case-sensitive name as the bundle itself
  - `asset_type_descriptor.xml`: the XML that describes the bundle (ie. all the classes, their relationships / containment, etc)
  - `i18n`: a directory containing labels
    - `labels.properties`: the translate-able set of class and attribute labels
  - `icons`: a directory containing two `.gif` files per class defined by the bundle:
    - `<ClassId>-icon.gif`: should be a 16x16 pixel representation of the class, and could also be a `.png` file, but should be named `.gif` regardless
    - `<ClassId>-bigIcon.gif`: should be a 32x32 pixel representation of the class, and could also be a `.png` file, but should be named `.gif` regardless

### `ibm_infosvr_openigc_asset_instances`

The list of XML files provided through this variable should be fully-working asset instance XML files, either already generated (eg. by variable below) or manually created.

### `ibm_infosvr_openigc_assets_as_yaml`

The list of YAML files provided through this variable are used to generate valid asset instance XMLs that are then loaded automatically. The structure of each YAML file should be as follows:

This variable is provided as a more convenient / readable way to specify asset instances than learning the XML format required by `ibm_infosvr_openigc_asset_instances` above.

```yml
---

bundleId: <BundleId>
contains:
  - class: <ClassName>
    name: <name>
    contains:
      - class: <NestedClassName>
        name: <name>
        contains:
          ...
```

The `contains` substructure can be nested as many times as necessary to create the containment hierarhcy you require for your asset(s). Each sub-structure must have at a minimum the `class` and `name` defined.

In addition, any number of additional attributes can also be specified as permitted by your bundle definition and the default set of attributes for all objects (eg. `short_description`, `long_description`). Any bundle-defined attributes simply need to be prefixed with `$`. For example:

```yml
---

bundleId: TestBundle
contains:
  - class: Folder
    name: level1
    short_description: The top-level directory
    $filesystem: UNIX
    contains:
      - class: Folder
        name: level2
        short_description: The next level down in the directory structure
        contains:
          - class: File
            name: MyFile.txt
            short_description: A file that sits within /level1/level2/MyFile.txt
            $encoding: UTF-8
```

In this example the `TestBundle` bundle defines `Folder`s and `File`s, where `Folder`s can have a `filesystem` attribute defined and `File`s can have an `encoding` defined. Both can have `short_description`s because all objects in IGC have this attribute.

For attributes that allow multiple values, simply specify define the value for the attribute as a YAML list. In this example, the `users_with_access` attribute specific to the bundle allows multiple values, so we specify each value as part of a YAML list for that attribute:

```yml
---

bundleId: TestBundle
contains:
  - class: Folder
    name: root
    $users_with_access:
      - user123
      - user456
      - user789
```

Finally, the `names` shortcut exists for leaf-nodes of a containment hierarchy where all you need to specify is the `name` of each one -- this avoids needing to specify the same `class` over and over again for each leaf node:

```yml
---

bundleId: TestBundle
contains:
  - class: Folder
    name: root
    contains:
      - class: File
        names:
          - MyFile.txt
          - YourFile.txt
          - OtherFile.txt
```

## License

Apache 2.0

## Author Information

Christopher Grote

![Ansible Role](https://img.shields.io/ansible/role/d/40612)

# ahuffman.satellite6_manage_content_views

An Ansible role to manage publishing and promotion of Satellite6 content views and composite content views.

## Role Variables
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| sat6_fqdn | yes | Fully-qualified domain name of your Satellite6 server. | "" | string |
| sat6_user | yes | Satellite6 user with rights to view/read/delete/publish/promote content views. | "" | string |
| sat6_pass | yes | Password of the `sat6_user`. Recommended to store this value encrypted in an Ansible vault.| "" | string |
| sat6_organization | yes | Name of the Satellite6 Organization where the content views reside. | "" | string |
| sat6_max_task_retries | no | Maximum number of attempts at checking a task's status in Satellite 6. Used with `sat6_retry_delay`| "240" | integer |
| sat6_retry_delay | no | Time to delay in seconds between retrying Satellite6 task status checks.  Used with `sat6_max_task_retries`. For example, `sat6_max_task_retries * sat6_retry_delay / 60 = 120` (1 hour).  This is how long we will wait for a publish, promote, create, delete of a content view to complete.| "30" | integer |
| sat6_content_views | yes | List of content views to act upon with the role. See the `sat6_content_views - Dictionary fields` section for details.| [] | list of dictionaries |
| sat6_content_view_show_stats | no | Whether or not to display a summary of actions the role completed on your content views | True | boolean |

### sat6_content_views - Dictionary fields
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| name | yes | Name of the content view to manage. | N/A | string |
| create_on_missing | no | *Not implemented yet* | False | boolean |
| publish_new_version | no | Publishes a new content view version if `True`. | False | boolean |
| publish_description | no | Description of content view changes during publishing of a new version. When used with a composite content view the underlying content views will be published with this description as well if using `components.publish_all: True`.| "" | string |
| publish_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while publishing a new content view version | False | boolean |
| promote_to | no | List of lifecycle environment/paths to promote the newest content view to.  We will skip promoting to any listed environments where the content view has already been promoted. | [] | list |
| promote_description | no | Description of content view promotion changes during promotion of a content view. When used with a composite content view the underlying content views will be promoted with this description as well if using `components.publish_all: True`.| "" | string |
| promote_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while promoting a content view to a lifecycle environment/path. | False | boolean |
| promote_remove_previous_version | no | Whether or not to remove the previous content view version when promoting a new version of a content view. This requires that you have already promoted versions to all lifecycle environments where the previous version had been promoted (i.e. the previous version cannot still be attached to lifecycle environments).  This restriction is the same whether you utilize the Satellite6 API, UI, or CLI.| False | boolean |
| keep_content_view_versions_count | no | Number of content view versions to keep.  ***Do not use when specifying `remove_content_view_versions` or `promote_remove_previous_version` as it will probably fail or create semi-unpredictable results.*** | '' | integer |
| promote_bypass_environment_path | no | Force promotion to a lifecycle environment/path outside of normal path restrictions (i.e. skip previous paths/environments) | False | boolean |
| remove_content_view_versions | no | Remove list of specified content view versions, which correspond to the version number ("Version" column of the Satellite6 content view's, "Versions" tab table).  This requires that the requested versions to delete are not associated with any lifecycle environments presently, including `Library`, which is a Satellite6 requirement. If you would like to delete a content view version associated with the `Library` lifecycle environment, remove it from that environment first.| [] | list |
| components | yes | Settings for composite content views.  ***Required only when*** working with composite content views | n/a | dictionary |

### sat6_content_views.components - Dictionary fields
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| publish_all | yes | Whether or not to publish new versions of all content views in a composite content view | False | boolean |
| content_views | no | List of content view version names and versions to publish the composite content view with | n/a | list of dictionaries |

### sat6_content_views.components.content_views - Dictionary fields
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| name | yes | Name of the content view in the composite content view | n/a | string |
| version | yes | Version number of the content view in the composite content view to publish the composite content view with | n/a | string |

### Special Statistical Reporting Variable
| Variable Name | Description | Type |
| --- | --- | --- |
| sat6_content_view_stats | A summary of all the actions completed by this role.  This allows for tasks outside of this role, such as a task to send an email notification or log results elsewhere to obtain the summary values. | list |

## Playbook Example Use-Cases
```yaml
---
- name: "Automated Satellite6 content view publish and promote"
  hosts: "localhost"
  gather_facts: False
  tasks:
    - name: "Manage Content Views"
      vars_files:
        - "vars/secrets.yml"
      include_role:
        name: "ansible-satellite6_manage_content_views"
      vars:
        sat6_fqdn: "huff-satellite.huffnet.org"
        sat6_user: "admin"
        sat6_pass: "{{ vaulted_sat6_pass }}"
        sat6_organization: "Huffnet"
        sat6_content_views:
          - name: "RHEL7"
            publish_new_version: True
            publish_description: "Latest content from Red Hat CDN"
            promote_to:
              - "RHEL7-Prod"
            promote_force_yum_metadata_regeneration: True
            promote_bypass_environment_path: True
          # Publish new version, promote to all lifecycle environments, remove old content view version
          - name: "RHEL6"
            publish_new_version: True
            promote_to:
              - "RHEL6-Dev"
              - "RHEL6-QA"
              - "RHEL6-Prod"
            promote_remove_previous_version: True
          # Remove specified content view versions
          - name: "RHEL7-HA Clustering"
            remove_content_view_versions:
              - "1.0"
              - "2.0"
              - "3.0"
          # publish new versions of all content views in a composite view and don't remove previous composite content view version, and promote to Dev
          - name: "RHEL7 composite"
            publish_new_version: True
            components:
              publish_all: True
            promote_to:
              - "RHEL7-Dev"
          # publish specific content view versions in a composite content view with publish and promote descriptions, remove previous composite content view version, and promote to 2 lifecycle environments
          - name: "RHEL7 composite2"
            publish_new_version: True
            publish_description: "Published by Ansible"
            promote_description: "Promoted by Ansible"
            promote_remove_previous_version: True
            components:
              publish_all: False
              content_views:
                - name: "RHEL7"
                  version: "3.0"
                - name: "RHEL7-HA Clustering"
                  version: "5.0"
            promote_to:
              - "RHEL7-Dev"
              - "RHEL7-QA"
          # Keep only 2 content view versions and remove the rest (can be used with promote and publish features if desired)
          - name: "RHEL7"
            keep_content_view_versions_count: "2"
```

## License
[MIT](LICENSE)

## Author Information
[Andrew J. Huffman](https://github.com/ahuffman)

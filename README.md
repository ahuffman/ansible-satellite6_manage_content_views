# ahuffman.satellite6_manage_content_views

***Work in progess***  
An Ansible role to manage creation, publishing, promotion, and deletion of Satellite6 content views.

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


### sat6_content_views - Dictionary fields
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| name | yes | Name of the content view to manage. | N/A | string |
| create_on_missing | no | *Not implemented yet* | False | boolean |
| publish_new_version | no | Publishes a new content view version if `True`. | False | boolean |
| publish_description | no | Description of content view changes during publishing of a new version | "" | string |
| publish_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while publishing a new content view version | False | boolean |
| promote_to | no | List of lifecycle environment/paths to promote the newest content view to.  We will skip promoting to any listed environments where the content view has already been promoted. | [] | list |
| promote_description | no | Description of content view promotion changes during promotion of a content view. | "" | string |
| promote_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while promoting a content view to a lifecycle environment/path. | False | boolean |
| promote_remove_previous_version | no | Whether or not to remove the previous content view version when promoting a new version of a content view. This requires that you have already promoted versions to all lifecycle environments where the previous version had been promoted (i.e. the previous version cannot still be attached to lifecycle environments).  This restriction is the same whether you utilize the Satellite6 API, UI, or CLI.| False | boolean |
| promote_bypass_environment_path | no | Force promotion to a lifecycle environment/path outside of normal path restrictions (i.e. skip previous paths/environments) | False | boolean |
| remove_content_view_versions | no | Remove list of specified content view versions, which correspond to the version number ("Version" column of the Satellite6 content view's, "Versions" tab table).  This requires that the requested versions to delete are not associated with any lifecycle environments presently, including `Library`, which is a Satellite6 requirement. If you would like to delete a content view version associated with the `Library` lifecycle environment, remove it from that environment first.| [] | list |

## Example Playbook
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
```

## License
[MIT](LICENSE)

## Author Information
[Andrew J. Huffman](https://github.com/ahuffman)

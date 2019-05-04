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
| sat6_content_views | yes | List of content views to act upon with the role. See the `sat6_content_views - Dictionary fields` section for details.| [] | list of dictionaries |


### sat6_content_views - Dictionary fields
| Variable Name | Required | Description | Default Value | Type |
| --- | :---: | --- | :---: | --- |
| name | yes | Name of the content view to manage. | N/A | string |
| create_on_missing | no | *Not implemented yet* | False | boolean |
| publish_new_version | no | Publishes a new content view version if `True`. | False | boolean |
| publish_description | no | Description of content view changes during publishing of a new version | "" | string |
| publish_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while publishing a new content view version | False | boolean |
| promote_to | no | List of lifecycle environment/paths to promote the content view to.  We will skip any listed environments where the content view has already been promoted.| [] | list |
| promote_description | no | Description of content view promotion changes during promotion of a content view. | "" | string |
| promote_force_yum_metadata_regeneration | no | Whether or not to force yum metadata regeneration while promoting a content view to a lifecycle environment/path. | False | boolean |
| promote_remove_previous_version | no | Remove the previous content view version when promoting a new version of a content view. This requires that you have already promoted versions to all lifecycle environments where the previous version had been promoted.  This restriction is the same whether you utilize the Satellite6 API, UI, or CLI.|
| promote_bypass_environment_path | no | Force promotion to a lifecycle environment/path outside of normal path restrictions (i.e. skip previous paths/environments) | False | boolean |

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
            create_on_missing: True
            publish_new_version: True
            publish_description: "Latest content from Red Hat CDN"
            publish_force_yum_metadata_regeneration: False
            promote_to:
              - "RHEL7-Dev"
              - "RHEL7-QA"
              - "RHEL7-Prod"
            promote_description: "Promoted by Ansible"
            promote_force_yum_metadata_regeneration: False
            promote_bypass_environment_path: True
          - name: "RHEL6"
            create_on_missing: True
            publish_new_version: True
            publish_description: "Latest content from Red Hat CDN"
            publish_force_yum_metadata_regeneration: False
            promote_to:
              - "RHEL6-Dev"
              - "RHEL6-QA"
              - "RHEL6-Prod"
            promote_description: "Promoted by Ansible"
            promote_force_yum_metadata_regeneration: False
            promote_bypass_environment_path: True
```

## License
[MIT](LICENSE)

## Author Information
[Andrew J. Huffman](https://github.com/ahuffman)

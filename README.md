# aspects_file_tasks
Generic file related tasks. A catch all for any file related task that doesn't need to be part of its own role. Currently can handle uploading files, deleting files, setting file/directory/symlink/hard link state, uploading Certificate Authority files, templating plain text files, and templating yaml formatted files.

See [Ansible's file module docs](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html) for details on how the `state`,`path`,`mode`,`owner`, and `group` parameters in the following docs work.


## Role Variables
### aspects_file_tasks_enabled
Default: `False`. Set to `True` to run the roles tasks.

Whether or not to include the tasks from this role.

### aspects_file_tasks_set_file_state_enabled
Default: `False`. Set to `True` to run the associated tasks.

Whether or not to include the tasks from `set_file_state.yml`.

### aspects_file_tasks_set_file_contents_enabled
Default: `False`. Set to `True` to run the associated tasks.

Whether or not to include the tasks from `set_file_contents.yml`.

### aspects_file_tasks_set_file_contents_cas_enabled
Default: `False`. Set to `True` to run the associated tasks.

Whether or not to include the tasks from `set_file_contents_cas.yml`.

### aspects_file_tasks_set_file_state_items
A dictionary/hash of files, directories, or links to be checked for a given state.

Use the following pattern:

```yaml
aspects_file_tasks_set_file_state_items:
  < item key >:
    enabled: < True|False >
    state: "< file|directory|touch|link|hard|absent >"
    dest: "< path/to/target >"
    mode: "< mode >"
    owner: "< owner >"
    group: "< group >"
```
### aspects_file_tasks_set_file_contents_uploads
A dictionary/hash of files to upload from your local system to the target system.

Use the following pattern:

```yaml
aspects_file_tasks_set_file_contents_uploads:
  < item key >:
    enabled: < True|False >
    src: "< path/to/file/to/be/uploaded >"
    dest: "< path/to/the/files/destination >"
    mode: "< mode >"
    owner: "< owner >"
    group: "< group >"
```
### aspects_file_tasks_set_file_contents_template_plain_files
A dictionary/hash of files to create or update with the contents given in the `block` parameter.

Use the following pattern:

```yaml
aspects_file_tasks_set_file_contents_template_plain_files:
  < item key >:
    enabled: < True|False >
    target_path: "< path/to/target >"
    mode: "< mode >"
    owner: "< owner >"
    group: "< group >"
    block: |
      < file contents >
```
### aspects_file_tasks_set_file_contents_template_yaml_files
A dictionary/hash of files to create or update with the contents given in the `block` parameter. The `block` parameter is yaml formatted and the file will be as well.

Use the following pattern:

```yaml
aspects_file_tasks_set_file_contents_template_yaml_files:
  < item key >:
    enabled: < True|False >
    target_path: "< path/to/target >"
    mode: "< mode >"
    owner: "< owner >"
    group: "< group >"
    block:
      < yaml >:
      	< goes >: here
```

### aspects_file_tasks_set_line_contents
A dictionary/hash of lines to create in arbitrary files.

Use the following pattern:

```yaml
aspects_file_tasks_set_line_contents:
  < item key >:
    enabled: < True|False >
    state: "< present|absent >"
    path: "< /path/to/file >"
    line: 'line contents'
```
> WARNING: The line is whitespace sensitive. If your line starts with 3 spaces, and the actual contents of the line in the target file exists with 4 spaces, then the line with 3 spaces will be added to the end of the file.

### aspects_file_tasks_set_file_contents_cas
A dictionary/hash of Certificate Authority files in `.pem` format to upload from your local system to your target system. 

> Note: On RedHat based systems, the target directory should likely be `/etc/pki/ca-trust/source/anchors/`. See the `man` page of `update-ca-trust` for details.
> On Debian based systems , the target directory should likely be `/usr/local/share/ca-certificates`. See the `man` page of `update-ca-certificates` for details.

Use the following pattern:

```yaml

aspects_file_tasks_set_file_contents_cas:
  < item key >:
    enabled: < True|False >
    src: "< path/to/file/to/be/uploaded >"
    dest: "< path/to/the/files/destination >"
    mode: "< mode >"
    owner: "< owner >"
    group: "< group >"
```

## Example Playbook

```yaml
---
- hosts:
  - test
  vars:
    aspects_file_tasks_enabled: True
    aspects_file_tasks_set_file_state_enabled: True
    aspects_file_tasks_set_file_contents_enabled: True
    aspects_file_tasks_set_file_contents_cas_enabled: True
    aspects_file_tasks_set_file_contents_uploads:
      itemalpha:
        enabled: True
        src: "{{ inventory_dir }}/files/exampleupload.txt"
        dest: "/root/exampleupload.txt"
        mode: "0640"
        owner: "root"
        group: "root"
    aspects_file_tasks_set_file_contents_template_plain_files:
      itemplain:
        enabled: True
        target_path: "/root/plaintext.txt"
        owner: "root"
        group: "root"
        mode: "0644"
        block: |
          This is a test file.
    aspects_file_tasks_set_file_contents_template_yaml_files:
      itemyaml:
        enabled: True
        target_path: "/root/examplefile.yaml"
        owner: "root"
        group: "root"
        mode: "0644"
        block:
          fruit:
            - oranges
            - apples
            - pears
          plants:
            trees:
              - pine
              - apple
            crops:
              - wheat
              - barley
    aspects_file_tasks_set_file_state_items:
      etcprofile:
        enabled: True
        state: "file"
        path: "/etc/profile"
        owner: "root"
        group: "root"
        mode: "0644"
      roottestdir:
        enabled: True
        state: "directory"
        path: "/root/testdir"
        owner: "root"
        group: "root"
        mode: "0755"
      roottouched:
        enabled: True
        state: "touch"
        path: "/root/touched"
        owner: "root"
        group: "root"
        mode: "0664"
      rootoptlink:
        enabled: True
        state: "link"
        src: "/opt"
        dest: "/root/opt"
        owner: "root"
        group: "root"
        mode: "0755"
      rootetcprofilehard:
        enabled: True
        state: "hard"
        src: "/etc/profile"
        dest: "/root/etcprofile"
        owner: "root"
        group: "root"
        mode: "0755"
      tmpabsent:
        enabled: True
        state: "absent"
        dest: "/tmp/absent"
    aspects_file_tasks_set_file_contents_cas:
      digicertca:
        enabled: True
        delete: False
        src: "{{ inventory_dir }}/files/certs/DigiCertCA.crt"
        dest: "{% if ansible_os_family == 'Debian' %}/usr/local/share/ca-certificates/DigiCertCA.crt{% else %}/usr/share/pki/ca-trust-source/anchors/DigiCertCA.pem{% endif %}"
        mode: "0644"
        owner: "root"
        group: "root"

  roles:
    - aspects_file_tasks
```
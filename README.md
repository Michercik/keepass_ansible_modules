# What is this
Custom ansible modules for managing local keepass database. They allow you to create or remove: database, keyfile, groups, entries, as well as getting entry details (title, username, password, etc.).

# Prerequisites
You need to make sure that you have installed all python libraries defined in [requirements.txt](requirements.txt):

```bash
pip install -r requirements.txt
```
You also need to copy needed modules to your ansible `library` directory and file [keepass_func.py](module_utils/keepass_func.py) to your ansible `module_utils` directory.

# Modules
| Name | Short description |
|------|-------------|
| [`keepass_create_db`](library/keepass_create_db.py)| Creates keepass database. |
| [`keepass_create_entry`](library/keepass_create_entry.py)| Creates entry (and groups if needed) in keepass database. |
| [`keepass_create_group`](library/keepass_create_group.py)| Creates emptry group in keepass database. |
| [`keepass_create_keyfile`](library/keepass_create_keyfile.py)| Creates keyfile. |
| [`keepass_get_entry`](library/keepass_get_entry.py)| Gets entry details from keepass database. |
| [`keepass_remove_entry`](library/keepass_remove_entry.py)| Removes entry from keepass database. |
| [`keepass_remove_group`](library/keepass_remove_group.py)| Removes group from keepass database. |

#### WARNING! Be mindful to not expose your secrets when using [keepass_get_entry](library/keepass_get_entry.py) module.

More detailed documentation about each module is inside its file.

## Example playbook

```yaml
- name: KeePass test playbook
  hosts: localhost
  
  vars:
    keepass_db_path: "/tmp/test_db.kdbx"
    keepass_db_password: "secret-password"
    keepass_db_keyfile: "/tmp/test-keyfile.keyx"
    keepass_db_users:
      - name: test_root
        username: test-user
        pass: test-pa
        expiry_time: "2024-09-30 12:30:00+02:00"
      - name: test_1
        username: test-user1
        pass: test-pass1
        group: 'test-group/subgroup1'
        expiry_time: "2024-09-30 12:30:00+02:00"
      - name: test_2
        username: test-user2
        pass: test-pass2
        group: 'test-group/subgroup2/subsub'


  tasks:
    - name: Create keyfile
      keepass_create_keyfile:
        path: "{{ keepass_db_keyfile }}"

    - name: Create KeePass database
      keepass_create_db:
        path: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"

    - name: Create entries
      keepass_create_entry:
        database: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"
        entry:
          title: "{{ item.name }}"
          username: "{{ item.username }}"
          password: "{{ item.pass }}"
          group: "{{ item.group | default(None) }}"
          expiry_time: "{{ item.expiry_time | default(None) }}"
      loop: "{{ keepass_db_users }}"


    - name: Remove entries
      keepass_remove_entry:
        database: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"
        entry: "{{ item.name }}"
        group: "{{ item.group | default(omit) }}"
      ignore_errors: true
      loop:
        - name: test_2
          group: test-group/subgroup2

    - name: Create groups
      keepass_create_group:
        database: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"
        group: "{{ item }}"
      loop:
        - cr_group1
        - cr_group2/subcr_group2

    - name: Remove groups
      keepass_remove_group:
        database: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"
        group: "{{ item }}"
      loop:
        - cr_group1
        - cr_group2

    - name: Get details of entries
      keepass_get_entry:
        database: "{{ keepass_db_path }}"
        password: "{{ keepass_db_password }}"
        keyfile: "{{ keepass_db_keyfile }}"
        name: "{{ item }}"
      loop:
        - test_root
        - test_1
        - test_2
      register: entries

    - name: Print details
      debug:
        msg: "{{ item.entry }}"
      loop: "{{ entries.results }}"
      loop_control:
        label: "{{ item.entry.title }}"
```
Fact caching (enable in ansible.cfg)
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400


Add
cacheable: yes
to playbook set_fact task


Later play
---
- name: "Use cached initial_packets in a later play"
  hosts: cisco_switch
  gather_facts: yes   # or set fact caching to be loaded
  tasks:
    - name: "Display cached initial_packets value"
      debug:
        msg: "Cached initial_packets: {{ hostvars[inventory_hostname].initial_packets }}"

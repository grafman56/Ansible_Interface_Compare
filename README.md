# Ansible: Compare Interface Counters Over Time

Snapshot a Cisco interface's `packets input` counter, wait, then compare. A small
demonstration of capturing device state with Ansible and persisting it between plays.
Two persistence approaches are shown: Ansible fact caching, and a plain file.

## Playbooks

| File | What it does |
|---|---|
| `interfacecomparev1.yml` | Single run: snapshot the counter, pause 30s, snapshot again, report whether it increased. |
| `initialstore.yml` | Capture the counter and write it to `/tmp/initial_packets.txt`. |
| `Readingstoredvalue.yml` | Read that stored value back later and compare it against the current counter. |

The split (`initialstore` + `Readingstoredvalue`) shows how to carry state across
*separate* runs, which a single play cannot do on its own.

## Requirements

    ansible-galaxy collection install cisco.ios ansible.netcommon

An inventory group named `cisco_switch` pointing at a reachable IOS device, with
`network_cli` connection vars set (user/password via `ansible-vault` or a prompt).

## Run

    ansible-playbook interfacecomparev1.yml        # one-shot before/after
    ansible-playbook initialstore.yml              # save a baseline
    # ...later...
    ansible-playbook Readingstoredvalue.yml        # compare against the baseline

## Persisting state the "real" way: fact caching

The file approach is simple but crude. For production, Ansible's built-in fact cache
survives between plays without touching the filesystem yourself. In `ansible.cfg`:

    [defaults]
    fact_caching = jsonfile
    fact_caching_connection = /tmp/ansible_facts
    fact_caching_timeout = 86400

Mark the fact cacheable when you set it (`set_fact:` with `cacheable: yes`), and a
later play with `gather_facts: yes` can read
`hostvars[inventory_hostname].initial_packets` directly.

## Notes

- The counter is parsed from raw `show interfaces` text. That is deliberate here to
  keep it dependency-free; on modern IOS-XE you would usually prefer structured
  `ios_facts`.
- The interface (`gi0/1`) is hardcoded for the demo. Make it a variable to reuse it.

# Elasticsearch Curator Ansible role

Ansible role for installing and configuring elasticsearch curator tool and
rules.

Role Variables
--------------

```yaml
---
elastic_curator_packages:
  - elasticsearch-curator
  # Needed for python3 click compatibility
  - locales-all

elastic_curator_version: 5

# Elastic's PGP key for signing their repository
elastic_curator_pgp_key: "46095ACC8548582C1A2699A9D27D666CD88E42B4"
elastic_curator_keyserver: pgp.mit.edu

# Elastic's curator debian repository
elastic_curator_elastic_repo_url: "deb [arch=amd64] http://packages.elastic.co/curator/{{ elastic_curator_version }}/debian stable main"

elastic_curator_etc_dir: /etc/curator

elastic_curator_esserver: "127.0.0.1"
elastic_curator_esserver_port: 9200

# Only applicable if the elastic_curator_action var is unmodified and/or
# added-back in upon modification
elastic_curator_delete_days: 45
elastic_curator_delete_prefix: "logstash-"

elastic_curator_config_client:
  hosts:
    - "{{ elastic_curator_esserver }}"
  port: "{{ elastic_curator_esserver_port }}"
  use_ssl: False
  ssl_no_validate: False
  timeout: 30
  master_only: False

elastic_curator_config_logging:
  loglevel: INFO
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']

elastic_curator_config_yaml:
  client: "{{ elastic_curator_config_client }}"
  logging: "{{ elastic_curator_config_logging }}"

# Default action to delete indices
# www.elastic.co/guide/en/elasticsearch/client/curator/5.1/ex_delete_indices.html
elastic_curator_action:
  1:
    action: delete_indices
    description: >-
      Delete indices older than {{ elastic_curator_delete_days }} days (based
      on index name), for {{elastic_curator_delete_prefix}} prefixed indices.
      Ignore the error if the filter does not result in an actionable list of
      indices (ignore_empty_list) and exit cleanly.
    options:
      ignore_empty_list: True
      disable_action: False
    filters:
      - filtertype: pattern
        kind: prefix
        value: "{{ elastic_curator_delete_prefix }}"
      - filtertype: age
        source: name
        direction: older
        timestring: '%Y.%m.%d'
        unit: days
        unit_count: "{{ elastic_curator_delete_days }}"
```

Example Playbook
----------------

```
---
- hosts: all
  roles:
    - role: freedomofpress.elastic-curator
```

Running the tests
-----------------

* TODO - Add a elasticsearch CI test :)

This role uses [Molecule] and [Testinfra] for testing. To use it:

```
pip install -r requirements.txt
molecule test
```

You can also run selective commands:

```
molecule idempotence
molecule verify
```

To fire up an elasticsearch UI for debugging, run:

```bash
make elastic-ui
```

See the [Molecule] docs for more info.

License
-------

MIT

[Molecule]: http://molecule.readthedocs.org/en/master/

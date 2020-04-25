特定のBaremetal nodeとRole(compute/controller)の紐づけ
==================================================================

Refer: https://docs.openstack.org/project-deploy-guide/tripleo-docs/latest/provisioning/node_placement.html

例えば、baremetal nodeが以下のようになっているとき
```
(undercloud) [stack@undercloud ~]$ openstack baremetal node list
+--------------------------------------+--------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name   | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+--------+---------------+-------------+--------------------+-------------+
| b9dfcca7-98a5-4328-b20b-47b975f41736 | node-a | None          | power off   | available          | False       |
| 22bd2c77-eb7f-448a-874f-12ca29ca3b97 | node-b | None          | power off   | available          | False       |
+--------------------------------------+--------+---------------+-------------+--------------------+-------------+
```
node-a -> controller, node-b -> compute のように決め打ちする


* baremetal nodeにcapability を設定
```
$ openstack baremetal node set <id> --property capabilities='node:controller-0'
$ openstack baremetal node set <id> --property capabilities='node:compute-0'
```
※ 必ず0から始める

* scheduler_hints_env.yaml をでRoleとbaremtal nodeを紐づける
```
--- 記載内容 ---
parameter_defaults:
    ControllerSchedulerHints:
        'capabilities:node': 'controller-%index%'
    ComputeSchedulerHints:
        'capabilities:node': 'compute-%index%'
--- ここまで　---
```        

* ついでにホスト名の指定も可能 (hostname_map.yamlとする)
```
--- 記載内容 ---
parameter_defaults:
  HostnameMap:
    %stackname%-controller-0: controller
    %stackname%-novacompute-0: compute001
--- ここまで ---
```

※ 左辺はroleに定義している HostnameFormatDefault の値になるのでこちらを見ておくこと  
   (computeはnovacomputeなので間違えやすい)
```
$ grep HostnameFormatDefault /usr/share/openstack-tripleo-heat-templates/roles_data.yaml
  HostnameFormatDefault: '%stackname%-controller-%index%'
  HostnameFormatDefault: '%stackname%-novacompute-%index%'  
```

* 作成したyamlを指定してdeployすれば出来上がり
```
$ openstack overcloud deploy --template \
    -e scheduler_hints_env.yaml \
    -e hostname_map.yaml
```




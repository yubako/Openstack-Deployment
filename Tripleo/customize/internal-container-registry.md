
overcloud の imageをundercloudから取得する
=======================================

Refer: https://docs.openstack.org/project-deploy-guide/tripleo-docs/latest/deployment/container_image_prepare.html#prepare-environment-containers


overcloudはcontainer imageをdefaultでinternetから取得するため、director上にregistryを  
用意し、overcloudはdirectorからimageを取得するようにする。  
この場合、overcloudがinternetに接続する必要はないので、Bridge Interfaceは不要になる。


Undercloud更新
----------------

* container用 parameterファイルを作成
```
$ openstack tripleo container image prepare default \
    --local-push-destination \
    --output-env-file ~/containers-prepare-parameter.yaml
```

* undercloud更新
   (次のcontainer image prepareを打てば、undercloud installは不要かもしれない)
```
$ vi undercloud.conf
---- 以下のように変更する ----
# REQUIRED if authentication is needed to fetch containers. This file
# should contain values for "ContainerImagePrepare" and
# "ContainerImageRegistryCredentials" that will be used to fetch the
# containers for the undercloud installation. `openstack tripleo
# container image prepare default` can be used to provide a sample
# "ContainerImagePrepare" value. Alternatively this file can contain
# all the required Heat parameters for the containers for advanced
# configurations. (string value)
container_images_file = containers-prepare-parameter.yaml   <= Uncommentして、作成したファイルを指定

# Used to add custom insecure registries for containers. (list value)
# Deprecated group/name - [DEFAULT]/docker_insecure_registries
container_insecure_registries = undercloud.localdomain:8787  <= Uncommentして、undercloudを指定する
--- ここまで ----

$ openstack undercloud install
```

* container image prepare
```
$ sudo openstack tripleo container image prepare \
    -r /usr/share/openstack-tripleo-heat-templates/roles_data.yaml \
    -e containers-prepare-parameter.yaml
```

* Centos7だとovercloudにpodmanが使えないみたいなのでdockerにするように変更
```
$ vi containers-prepare-parameter.yaml

parameter_defaults:
  ContainerImagePrepare:
  - push_destination: true
    set:
      ceph_alertmanager_image: alertmanager
      ceph_alertmanager_namespace: docker.io/prom
      ceph_alertmanager_tag: v0.16.2
      ceph_grafana_image: grafana
      ceph_grafana_namespace: docker.io/grafana
      ceph_grafana_tag: 5.2.4
      ceph_image: daemon
      ceph_namespace: docker.io/ceph
      ceph_node_exporter_image: node-exporter
      ceph_node_exporter_namespace: docker.io/prom
      ceph_node_exporter_tag: v0.17.0
      ceph_prometheus_image: prometheus
      ceph_prometheus_namespace: docker.io/prom
      ceph_prometheus_tag: v2.7.2
      ceph_tag: v4.0.10-stable-4.0-nautilus-centos-7-x86_64
      name_prefix: centos-binary-
      name_suffix: ''
      namespace: docker.io/tripleotrain
      neutron_driver: ovn
      rhel_containers: false
      tag: current-tripleo
    tag_from_label: rdo_version
  ContainerCli: docker  <-- ★ 追加
```

Overcloud Deploy更新
---------------------

```
$ openstack overcloud deploy --templates \
    -e containers-prepare-parameter.yaml
```

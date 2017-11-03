Ansible Role: Headphones for Kubernetes
=======================================

Ansible role to install Headphones on Kubernetes.

Role Variables
--------------

```yaml
# Namespace
kubernetes_headphones_namespace: "default"
# App name (used as selector)
kubernetes_headphones_app: "headphones"
# Deployment name
kubernetes_headphones_deployment: "headphones-deployment"
# Service name
kubernetes_headphones_service: "headphones"

# Number of replicas
kubernetes_headphones_replicas: 1
kubernetes_headphones_revision_history: 1

# Node selector
kubernetes_headphones_node_selector: {}

kubernetes_headphones_resources:
  limits:
    memory: "756Mi"
  requests:
    memory: "256Mi"

# Setup ingress for headphones
kubernetes_headphones_setup_ingress: false
kubernetes_headphones_ingress:
  name: "headphones-ingress"
  host: "headphones.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /downloads/ (see examples for more details)
kubernetes_headphones_downloads_volumes: {}
# Music volumes. Mounted in /music/ (see examples for more details)
kubernetes_headphones_music_volumes: {}
# Watch directories. Useful when blackhole is used. Mounted in /watchdirs/ (see
# examples for more details)
kubernetes_headphones_watchdirs_volumes: {}

# Enby config volume. Contains the database, arts cache and config.
kubernetes_headphones_config_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_headphones_downloads_volumes:
      # This volume will be mounted as /downloads/completed
      completed:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-torrents
            readOnly: false
        subPath: "completed"

    kubernetes_headphones_music_volumes:
      # This volume will be mounted as /music/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-music
            readOnly: false

    # Directories watched by our torrents client
    kubernetes_headphones_watchdirs_volumes:
      # This volume will be mounted as /watchdirs/providers
      providers:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-watchdirs
            readOnly: false
        subPath: "example/providers"

    kubernetes_headphones_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-headphones
          readOnly: false
        subPath: "config"

    kubernetes_headphones_setup_ingress: true
    kubernetes_headphones_ingress:
      name: "headphones-ingress"
      host: "headphones.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "headphones-ingress-tls"
          hosts:
            - "headphones.example.com"
  roles:
    - role: Anthony25.kubernetes-headphones
```

Use `run_once` to run the role on only one available master in the cluster.

If the headphones web interface is not reachable, please check that it is
listening on `0.0.0.0`, and not only `localhost`. Also check that the public
http port is setup as 8181.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!

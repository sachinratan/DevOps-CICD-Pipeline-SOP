## Following guide will help with how to change the Containerd root path and confirm that the Containerd can leverage the specified storage directory for root path.

1) Generated the default Containerd config:
```
$ sudo containerd config default > /etc/containerd/config.toml
```

2) Created the root path directories:
```
$ mkdir -p /data/containerd /data/containerd-state
```

3) Changed the root path in Containerd `config.toml`:
```
$ sed -i -e 's#^[#[:space:]]*root[[:space:]]*=.*#root = "/data/containerd"#' -e 's#^[#[:space:]]*state[[:space:]]*=.*#state = "/data/containerd
-state"#' /etc/containerd/config.toml || true
```

4) Confirmed the root path changes:
```
$ cat /etc/containerd/config.toml
version = 3
root = "/data/containerd"
state = "/data/containerd-state"
temp = ''
plugin_dir = ''
disabled_plugins = []
required_plugins = []
oom_score = 0
imports = []
... output omitted ...
```

5) Restarted the Containerd service:
```
$ sudo systemctl restart containerd
```
OR
```
$ for i in {1..30}; do if systemctl is-active --quiet containerd; then echo "containerd restarted successfully"; break; fi; sleep 2; done
containerd restarted successfully
```

6) Containerd service status check:
```
$ systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/usr/lib/systemd/system/containerd.service; disabled; preset: disabled)
     Active: active (running) since Sat 2025-10-04 09:00:38 UTC; 1min 31s ago
       Docs: https://containerd.io
    Process: 27521 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 27522 (containerd)
      Tasks: 20
     Memory: 23.7M
        CPU: 727ms
     CGroup: /system.slice/containerd.service
             ├─27385 /usr/bin/containerd-shim-runc-v2 -namespace moby -id e094ce42faxxxxxxx858c89a0f573dc15 -address /run/containerd/c>
             └─27522 /usr/bin/containerd
... output omitted ...
```

7) Deployed test container to confirm the Container uses the new root path:
```
# sudo ctr image pull docker.io/library/nginx:latest
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  fetching image content
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
docker.io/library/nginx:latest                  saved
└──index (8adbdcb969e2)                         complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (0546c3c7854a)                   waiting         |--------------------------------------|
   │  └──config (de844655ad84)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (17ae566734b6)                   waiting         |--------------------------------------|
   │  ├──config (203ad09fc156)                  complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (5c32499ab806)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (375a694db734)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (5f825f15e2e0)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (16d05858bb8d)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (08cfef42fd24)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  ├──layer (3cc5fdd1317a)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──layer (4f4e50e20765)                   complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (ae6fc6312bf3)                   waiting         |--------------------------------------|
   │  └──config (13a038f4d4d2)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (53802b6f8a42)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (9352a45c047b)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (dd176a1a03d1)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (d9e6952fa25a)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (e181c2faf986)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (3f1a15e039b8)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (e041cf856a0f)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (0777d15d89ec)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (ed906453ed37)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (2e7ce1c1c985)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (19a49ca07e70)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (67e9ccbacedc)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (6a55bf79c3ed)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (f5d7c9446e3f)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (e3c09c211ae2)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (6e9f8775283f)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (a93ad1d17f91)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (025802610d90)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (cf23757a83fc)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (65bd04318dda)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (e1b43bdef5cb)                   complete        |++++++++++++++++++++++++++++++++++++++|
   │  └──config (11e12aba8e15)                  complete        |++++++++++++++++++++++++++++++++++++++|
   ├──manifest (35d8313cc597)                   waiting         |--------------------------------------|
   │  └──config (2135552f4353)                  complete        |++++++++++++++++++++++++++++++++++++++|
   └──manifest (d31b68a082fb)                   complete        |++++++++++++++++++++++++++++++++++++++|
      └──config (6ff6725724ca)                  complete        |++++++++++++++++++++++++++++++++++++++|
application/vnd.oci.image.index.v1+json sha256:8adbdcb969e2676478ee2c7ad333956f0c8e0e4c5a7463f4611d7a2e7a7ff5dc
Pulling from OCI Registry (docker.io/library/nginx:latest)      elapsed: 5.1 s  total:  69.0 M  (13.5 MiB/s)
```
```
# sudo ctr run -d docker.io/library/nginx:latest test-nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
```
```
# sudo ctr task ls
TASK          PID      STATUS
test-nginx    30304    RUNNING
```
```
# ls -lrth /data/containerd/
io.containerd.content.v1.content/         io.containerd.runtime.v2.task/            io.containerd.snapshotter.v1.native/
io.containerd.grpc.v1.introspection/      io.containerd.sandbox.controller.v1.shim/ io.containerd.snapshotter.v1.overlayfs/
io.containerd.metadata.v1.bolt/           io.containerd.snapshotter.v1.blockfile/   tmpmounts/
```
```
# ls -lrth /data/containerd/io.containerd.snapshotter.v1.overlayfs/
total 72K
drwx------. 10 root root   78 Oct  4 09:03 snapshots
-rw-------.  1 root root 128K Oct  4 09:03 metadata.db
```
```
# ls -lrth /data/containerd/io.containerd.runtime.v2.task/
default/ moby/
```
```
# ls -lrth /data/containerd/io.containerd.runtime.v2.task/
total 0
drwx--x--x. 3 root root 24 Oct  4 09:03 default
drwx--x--x. 2 root root  6 Oct  4 09:05 moby
```
```
# ls -lrth /data/containerd/io.containerd.runtime.v2.task/default/            --> ##### The container directory created in new root path ####
test-nginx/
```
```
# ls -lrth /data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/
1/  10/ 2/  3/  4/  5/  6/  7/  8/  9/
```
```
# ls -lrth /data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/
total 0
drwx------. 4 root root 28 Oct  4 09:03 1
drwx------. 4 root root 28 Oct  4 09:03 2
drwx------. 4 root root 28 Oct  4 09:03 3
drwx------. 4 root root 28 Oct  4 09:03 4
drwx------. 4 root root 28 Oct  4 09:03 5
drwx------. 4 root root 28 Oct  4 09:03 6
drwx------. 4 root root 28 Oct  4 09:03 7
drwx------. 4 root root 28 Oct  4 09:03 8
drwx------. 4 root root 28 Oct  4 09:06 9
drwx------. 4 root root 28 Oct  4 09:06 10
```
```
# date
Sat Oct  x 09:07:43 UTC 2025
```
```
# ls -lrth /data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/1/fs/
bin/   boot/  dev/   etc/   home/  lib/   lib64/ media/ mnt/   opt/   proc/  root/  run/   sbin/  srv/   sys/   tmp/   usr/   var/ 
```

8) Verified that containers are not using the Containerd default old path location:
```
# ls -lrth /run/containerd/io.containerd.runtime.v2.task/default/
total 0
```

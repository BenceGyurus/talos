# Talos

## Init

```bash
cp .env.template .env
```

Then fill your data the .env and export with

```bash
set -a && source .env && set +a
```

To get your disk name run the command:

```bash
talosctl get disks --insecure --nodes $CONTROL_PLANE_IP
```

For example:
```bash
NODE   NAMESPACE   TYPE   ID      VERSION   SIZE     READ ONLY   TRANSPORT   ROTATIONAL   WWID   MODEL           SERIAL
       runtime     Disk   loop0   2         4.1 kB   true                                                        
       runtime     Disk   loop1   2         479 kB   true                                                        
       runtime     Disk   loop2   2         84 MB    true                                                        
       runtime     Disk   sda     2         32 GB    false       virtio      true                QEMU HARDDISK   
       runtime     Disk   sr0     2         338 MB   true        ata         true                QEMU DVD-ROM    QEMU_DVD-ROM_QM00003
```

My disk name is: sda

Then run the command below to generate your configuration:
```bash
talosctl gen config $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 --install-disk /dev/$DISK_NAME
```

It is being created a controlplane.yml and a worker.yml in that folder where you execute the command above.

You can apply the configuration on controlplane with:

```bash
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
```

and on the workeres:

```bash
for ip in "${WORKER_IP[@]}"; do
    echo "Applying config to worker node: $ip"
    talosctl apply-config --insecure --nodes "$ip" --file worker.yaml
done
```

Set the endpoints: (it is going to set the controlplane ip into your `talosconfig` file)

```bash
talosctl --talosconfig=./talosconfig config endpoints $CONTROL_PLANE_IP
```

And bootstrap your cluster:

```shell
talosctl bootstrap --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

 to merge your Kubernetes configurations:

```bash
talosctl kubeconfig --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

Usefull command:

health check:

```bash
talosctl --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig health
```

get talos dashboard:

```bash
talosctl --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig dashboard
```

## Flux commands

update the git repository in order ot pull the modifyed yaml:
```bash
flux reconcile source git flux-system
```

syncronize the kustiomizations:
```bash
flux reconcile ks portfolio
```

get the kustiomizations:
```bash
flux get kustomizations
```

remove kustomization
```bash
flux delete kustomization <name> --namespace flux-system
```
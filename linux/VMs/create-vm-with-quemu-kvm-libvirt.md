We need QUEMU/KVM and libvirt to create VMS.

# Useful Tools

-   Virtual Machine Manager
-   Cockpit

# Create Main Persistent QCOW2 Image

```sh
qemu-img create -f qcow2 kube-vm1.qcow2 10G
```

This is create an img which can be used to boot our VM. But this doesn't contain anything.

# Create Main Persistent QCOW2 Image with a BASE Image

We'll be using ubuntu. A normal ubuntu server or ubuntu cloud image can be used.

Example with a cloud image.

For downloading cloud images for ubuntu go https://cloud-images.ubuntu.com

Here, choose the version you wanna download and inside there search for images with `.img`. Choose according to your device's specification.

In this case we choose https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img.

```sh
qemu-img create -b your-downloaded-cloud-image.img -f qcow2 kube-vm1.qcow2 10G
```

This will create an img of 10GB with our base cloud image.

For better details check out https://sumit-ghosh.com/posts/create-vm-using-libvirt-cloud-images-cloud-init.

# Setting default login creds for the BASE Cloud Image

When we'll boot into the VM, it'll ask for user and pass which we never created. For this a better way is to create a meta-data where we can specify login creds for ssh access, networking, etc.

For better details check out https://sumit-ghosh.com/posts/create-vm-using-libvirt-cloud-images-cloud-init.

**But there's an easier way too, which is to set a default username and password for the base image.**

Later we can always create ssh keys and not rely on password login.

```sh
virt-customize -a your-downloaded-cloud-image.img --root-password password:<pass>
```

**This step needs to be done before launching the VM**

# Lanching our VM

```sh
virt-install --name=kube-1 --ram=512 --vcpus=1 --import --disk path=kube-1.img,format=qcow2 --os-variant=ubuntu22.04 --network bridge=br0,model=virtio --graphics vnc,listen=0.0.0.0 --noautoconsole
```

**Change OS and network config according to your settings.**

This will create a VM of 512MB memory, 1 VCPU Core, with out built image of 10GB, we also allow VNC to later manage our VM visually.

# Utilising all available storage out of 10GB for `/`

In my case with ubuntu22.04 LTS cloud image, the system only allocated 2.2GB of storage for `/` even though the total space was 10GB. To fix this,

First, run `lsblk`, it'll show you the available drives with their partitions.

In my case the `/` used only 2.2G but more than 7GB was unused.

-   Find the label or name which should be something like `/dev/vda`.

-   Then, run `sudo parted /dev/vda print` to see your partition structure.

-   Now verify filesystem type, run `sudo blkid /dev/vda1`. Here vda1 was my partion for `/`.

> **In my case there's no unallocated space showing, so I had to do this.**

-   Run `sudo parted /dev/vda`, and again run `resizepart 1 100%`

-   For `ext4`, run `sudo resize2fs /dev/vda1`. This should allow to take the rest of the available space.

Now, we should see `/` using the entire available space.

**Note: I'm no expert here, doing this to mimic a k3s high available infra in my homelab for learning. Pretty sure there's a better way to do this with less trouble, but currently I found this to be the straightforward for me.**

**Will update this if I found a better way.**

# Resources

Just search **cloud image vm in kvm libvirt**. The first couple of sites gives useful info.

# References

-   https://sumit-ghosh.com/posts/create-vm-using-libvirt-cloud-images-cloud-init
-   ChatGPT 😏

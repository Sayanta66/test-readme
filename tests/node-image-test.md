## Creating a viable Node Image

For using cluster-API with the bootstrap provider kubeadm, we need a server with all the necessary binaries and settings for running Kubernetes.
There are several ways to achieve this. In the quick-start guide, we use `pre-kubeadm` commands in the KubeadmControlPlane and KubeadmConfigTemplate objects. These are propagated from the bootstrap provider kubeadm and the control plane provider kubeadm to the node as cloud-init commands. This way is usable universally also in other infrastructure providers.
For Hcloud, there is an alternative way of doing this using Packer. It creates a snapshot to boot from. This makes it easier to version the images, and creating new nodes using this image is faster. The same is possible for Hetzner Bare Metal, as we could use installimage and a prepared tarball, which then gets installed.

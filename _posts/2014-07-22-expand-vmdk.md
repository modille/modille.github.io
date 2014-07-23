---
layout: post
title: HOWTO - Add space to a dynamically allocated VMDK
---
<h1> {{ page.title }} </h1>

I had a VM set up with a dynamically-allocated VMDK with a max size of 60GB. I reached near 100% capacity on the VMDK,
and used the following steps to grow the VMDK.

Note: YMMV; I did this using a Windows 8 host, Fedora 20 guest, and VirtualBox 4.3.12

# Create new VMDK {#create-new-vmdk}

<ol>
<li>Shut down your VM</li>
<li>Within VirtualBox Manager, open the Settings for your existing VM, then choose Storage</li>
<li>Add a hard disk to the controller. Make it the same type as your current disk, and make it the size you need</li>
<li>Once it’s created open a command prompt and head to the directory containing <code>vboxmanage.exe</code> – typically in <code>C:\Program Files\Oracle\VirtualBox\</code></li>
<li>Run the following, altering the paths to match your current and new VMDK files:
{% highlight bash %}
vboxmanage clonehd "C:\vm\fedora20-disk1.vmdk" "C:\vm\NewVirtualDisk.vmdk" --existing
{% endhighlight %}</li>
<li>Once that’s complete, reopen the VirtualBox Manager and head back to Settings > Storage</li>
<li>Add an existing hard disk to your controller, choosing your new drive</li>
<li>Disconnect your original drive</li>
<li>Select your new drive and set it to SATA Port 0</li>
</ol>

# Boot into live CD {#boot-into-live-cd}

I booted into a live CD so that my `/dev/sda2` partition wasn't mounted. This may have actually been optional, since
my partitions were setup with LVM.

1. Download Fedora Desktop live CD ISO
2. Mount it to your VM using `Devices > CD/DVD Devices > Choose...`
3. Restart your VM to boot into the live CD

# Resize the /dev/sda2 partition {#resize-dev-sda2-partition}

Again, you might be able to just use gparted, depending on your setup. Fedora 20 doesn't have any graphical partition
manager that I'm aware of, so I did the following:

<ol>
<li>Open a terminal and do the following:
{% highlight bash %}
$ sudo fdisk -l # View partition layout; typically we're dealing with /dev/sda2
$ sudo fdisk /dev/sda
  d # Delete a partition
  2 # If we want to grow /dev/sda2; NOTE: This does not delete any data on disk
  n # Create a new partition
  p # Primary partition
  2
  <return> # Default starting block
  <return> # Default ending block, full size of the partition

  # Make sure partition type is 8e for Linux LVM:
  t
  8e
  w # Write changes to disk
{% endhighlight %}</li>
<li>Restart your VM to get the new partition table</li>
</ol>

# Resize home volume {#resize-home-volume}

<ol>
<li>Again boot into the live CD, open a terminal, and do the following:

{% highlight bash %}
$ sudo pvresize /dev/sda2
$ sudo pvscan # Should show new larger size
$ sudo lvextend -l +100%FREE /dev/fedora/home
$ sudo e2fsck -f /dev/fedora/home
$ sudo resize2fs /dev/fedora/home
{% endhighlight %}</li>

<li>At this point, <code>df</code> should show the new space</li>
</ol>

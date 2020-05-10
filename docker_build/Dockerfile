FROM ubuntu:focal

# Docker image that builds a live bootable iso based on the above Ubuntu

# Install the ISO making tools that we need
RUN echo "APT::Acquire::Retries \"3\";" > /etc/apt/apt.conf.d/80-retries
RUN apt-get update && apt-get -y install dosfstools zip binutils debootstrap squashfs-tools xorriso grub-pc-bin grub-efi-amd64-bin mtools

# Create the chroot
RUN mkdir /live-ubuntu-from-scratch && debootstrap --arch=amd64 --variant=minbase focal /live-ubuntu-from-scratch/chroot http://us.archive.ubuntu.com/ubuntu/
# RUN mount --bind /dev $HOME/live-ubuntu-from-scratch/chroot/dev && mount --bind /run $HOME/live-ubuntu-from-scratch/chroot/run

# Run the system level chrootscript in the chroot
ADD chrootscript /live-ubuntu-from-scratch/chroot
RUN chmod a+x /live-ubuntu-from-scratch/chroot/chrootscript

WORKDIR /root

ADD grub.cfg /root
RUN chroot /live-ubuntu-from-scratch/chroot ./chrootscript

ADD NetworkManager.conf /live-ubuntu-from-scratch/chroot
ADD resolv.conf /live-ubuntu-from-scratch/chroot
ADD lightdm.conf /live-ubuntu-from-scratch/chroot

ADD README.diskdefines /root
ADD build_iso /root
RUN chmod a+x /root/build_iso

ADD run_chroot_scripts /root
RUN chmod a+x /root/run_chroot_scripts

ADD chroot_priv_script /live-ubuntu-from-scratch/chroot
RUN chmod a+x /live-ubuntu-from-scratch/chroot/chroot_priv_script

# Add the stubs
ADD stubs/chroot_custom_stub /live-ubuntu-from-scratch/chroot/chroot_custom
RUN chmod a+x /live-ubuntu-from-scratch/chroot/chroot_custom
ADD stubs/pre_custom_stub /root/pre_custom
RUN chmod a+x /root/pre_custom
ADD stubs/post_custom_stub /root/post_custom
RUN chmod a+x /root/post_custom


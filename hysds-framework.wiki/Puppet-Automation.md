##  Create a base CentOS 7 image for installation of all HySDS component instances
  1. Start up a CentOS 7 instance. Use image from http://cloud.centos.org/centos/7/images/ for private cloud provisioning. For public cloud provisioning use CentOS 7 x86_64 HVM image at https://wiki.centos.org/Cloud/AWS.
  1. Log into instance as user centos and sudo to root:
      ```
      sudo su -
      ```
  1. Disable SELinux by modifying `/etc/selinux/config`:
      ```
      # This file controls the state of SELinux on the system.
      # SELINUX= can take one of these three values:
      #     enforcing - SELinux security policy is enforced.
      #     permissive - SELinux prints warnings instead of enforcing.
      #     disabled - No SELinux policy is loaded.
      SELINUX=disabled
      # SELINUXTYPE= can take one of three two values:
      #     targeted - Targeted processes are protected,
      #     minimum - Modification of targeted policy. Only selected processes are protected.
      #     mls - Multi Level Security protection.
      SELINUXTYPE=targeted
      ```
  1. Install EPEL repo, update and upgrade:
      ```
      yum install -y epel-release
      yum update
      ```
  1. Install requisite yum packages:
      ```
      yum -y install puppet puppet-firewalld nscd ntp wget curl subversion git vim screen
      ```
  1. Clean yum cache:
      ```
      yum clean all
      ```
  1. Clean cloud-init data:
      ```
      rm -rf /var/lib/cloud/*
      ``` 
  1. Create an image of this instance using the method that pertains to your private/public cloud implementation. This image will now be used for generating all other HySDS component instances.
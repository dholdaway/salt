============================
Getting Started With AWS EC2
============================

Amazon EC2 is a very widely used public cloud platform and one of the core
platforms Salt Cloud has been built to support.

Previously, the suggested provider for AWS EC2 was the `aws` provider. This has
been deprecated in favor of the `ec2` provider. Configuration using the old
`aws` provider will still function, but that driver is no longer in active
development.

.. code-block:: yaml

    # Note: This example is for /etc/salt/cloud.providers or any file in the
    # /etc/salt/cloud.providers.d/ directory.

    my-ec2-southeast-public-ips:
      # Set up the location of the salt master
      #
      minion:
        master: saltmaster.example.com

      # Set up grains information, which will be common for all nodes
      # using this provider
      grains:
        node_type: broker
        release: 1.0.1

      # Specify whether to use public or private IP for deploy script.
      #
      # Valid options are:
      #     private_ips - The salt-master is also hosted with EC2
      #     public_ips - The salt-master is hosted outside of EC2
      #
      ssh_interface: public_ips

      # Set the EC2 access credentials (see below)
      #
      id: HJGRYCILJLKJYG
      key: 'kdjgfsgm;woormgl/aserigjksjdhasdfgn'

      # Make sure this key is owned by root with permissions 0400.
      #
      private_key: /etc/salt/my_test_key.pem
      keyname: my_test_key
      securitygroup: default

      # Optionally configure default region
      #
      location: ap-southeast-1
      availability_zone: ap-southeast-1b

      # Configure which user to use to run the deploy script. This setting is
      # dependent upon the AMI that is used to deploy. It is usually safer to
      # configure this individually in a profile, than globally. Typical users
      # are:
      #
      # Amazon Linux -> ec2-user
      # RHEL         -> ec2-user
      # CentOS       -> ec2-user
      # Ubuntu       -> ubuntu
      #
      ssh_username: ec2-user

      # Optionally add an IAM profile
      iam_profile: 'arn:aws:iam::123456789012:instance-profile/ExampleInstanceProfile'

      provider: ec2


    my-ec2-southeast-private-ips:
      # Set up the location of the salt master
      #
      minion:
        master: saltmaster.example.com

      # Specify whether to use public or private IP for deploy script.
      #
      # Valid options are:
      #     private_ips - The salt-master is also hosted with EC2
      #     public_ips - The salt-master is hosted outside of EC2
      #
      ssh_interface: private_ips

      # Set the EC2 access credentials (see below)
      #
      id: HJGRYCILJLKJYG
      key: 'kdjgfsgm;woormgl/aserigjksjdhasdfgn'

      # Make sure this key is owned by root with permissions 0400.
      #
      private_key: /etc/salt/my_test_key.pem
      keyname: my_test_key
      securitygroup: default

      # Optionally configure default region
      #
      location: ap-southeast-1
      availability_zone: ap-southeast-1b

      # Configure which user to use to run the deploy script. This setting is
      # dependent upon the AMI that is used to deploy. It is usually safer to
      # configure this individually in a profile, than globally. Typical users
      # are:
      #
      # Amazon Linux -> ec2-user
      # RHEL         -> ec2-user
      # CentOS       -> ec2-user
      # Ubuntu       -> ubuntu
      #
      ssh_username: ec2-user

      # Optionally add an IAM profile
      iam_profile: 'my other profile name'

      provider: ec2


Access Credentials
==================
The ``id`` and ``key`` settings may be found in the Security Credentials area
of the AWS Account page:

https://portal.aws.amazon.com/gp/aws/securityCredentials

Both are located in the Access Credentials area of the page, under the Access
Keys tab. The ``id`` setting is labeled Access Key ID, and the ``key`` setting
is labeled Secret Access Key.


Key Pairs
=========
In order to create an instance with Salt installed and configured, a key pair
will need to be created. This can be done in the EC2 Management Console, in the
Key Pairs area. These key pairs are unique to a specific region. Keys in the
us-east-1 region can be configured at:

https://console.aws.amazon.com/ec2/home?region=us-east-1#s=KeyPairs

Keys in the us-west-1 region can be configured at

https://console.aws.amazon.com/ec2/home?region=us-west-1#s=KeyPairs

...and so on. When creating a key pair, the browser will prompt to download a
pem file. This file must be placed in a directory accessable by Salt Cloud,
with permissions set to either 0400 or 0600.


Security Groups
===============
An instance on EC2 needs to belong to a security group. Like key pairs, these
are unique to a specific region. These are also configured in the EC2
Management Console. Security groups for the us-east-1 region can be configured
at:

https://console.aws.amazon.com/ec2/home?region=us-east-1#s=SecurityGroups

...and so on.

A security group defines firewall rules which an instance will adhere to. If
the salt-master is configured outside of EC2, the security group must open the
SSH port (usually port 22) in order for Salt Cloud to install Salt.


IAM Profile
===========
Amazon EC2 instances support the concept of an `instance profile`_, which
is a logical container for the IAM role. At the time that you launch an EC2
instance, you can associate the instance with an instance profile, which in
turn corresponds to the IAM role. Any software that runs on the EC2 instance
is able to access AWS using the permissions associated with the IAM role.

Scaffolding the profile is a 2 steps configurations:

 1. Configure an IAM Role from the `IAM Management Console`_.
 2. Attach this role to a new profile. It can be done with the `AWS CLI`_:

    .. code-block:: bash

      > aws iam create-instance-profile --instance-profile-name PROFILE_NAME
      > aws iam add-role-to-instance-profile --instance-profile-name PROFILE_NAME --role-name ROLE_NAME

Once the profile is created, you can use the **PROFILE_NAME** to configure
your cloud profiles.

.. _`IAM Management Console`: https://console.aws.amazon.com/iam/home?#roles
.. _`AWS CLI`: http://docs.aws.amazon.com/cli/latest/index.html
.. _`instance profile`: http://docs.aws.amazon.com/IAM/latest/UserGuide/instance-profiles.html


Cloud Profiles
==============
Set up an initial profile at ``/etc/salt/cloud.profiles``:

.. code-block:: yaml

    base_ec2_private:
      provider: my-ec2-southeast-private-ips
      image: ami-e565ba8c
      size: Micro Instance
      ssh_username: ec2-user

    base_ec2_public:
      provider: my-ec2-southeast-public-ips
      image: ami-e565ba8c
      size: Micro Instance
      ssh_username: ec2-user

    base_ec2_db:
      provider: my-ec2-southeast-public-ips
      image: ami-e565ba8c
      size: m1.xlarge
      ssh_username: ec2-user
      volumes:
        - { size: 10, device: /dev/sdf }
        - { size: 10, device: /dev/sdg, type: io1, iops: 1000 }
        - { size: 10, device: /dev/sdh, type: io1, iops: 1000 }


The profile can be realized now with a salt command:

.. code-block:: bash

    # salt-cloud -p base_ec2 ami.example.com
    # salt-cloud -p base_ec2_public ami.example.com
    # salt-cloud -p base_ec2_private ami.example.com


This will create an instance named ``ami.example.com`` in EC2. The minion that
is installed on this instance will have an ``id`` of ``ami.example.com``. If
the command was executed on the salt-master, its Salt key will automatically be
signed on the master.

Once the instance has been created with salt-minion installed, connectivity to
it can be verified with Salt:

.. code-block:: bash

    # salt 'ami.example.com' test.ping


Required Settings
=================
The following settings are always required for EC2:

.. code-block:: yaml

    # Set the EC2 login data
    my-ec2-config:
      id: HJGRYCILJLKJYG
      key: 'kdjgfsgm;woormgl/aserigjksjdhasdfgn'
      keyname: test
      securitygroup: quick-start
      private_key: /root/test.pem
      provider: ec2


Optional Settings
=================

EC2 allows a location to be set for servers to be deployed in. Availability
zones exist inside regions, and may be added to increase specificity.

.. code-block:: yaml

    my-ec2-config:
      # Optionally configure default region
      location: ap-southeast-1
      availability_zone: ap-southeast-1b


EC2 instances can have a public or private IP, or both. When an instance is
deployed, Salt Cloud needs to log into it via SSH to run the deploy script.
By default, the public IP will be used for this. If the salt-cloud command is
run from another EC2 instance, the private IP should be used.

.. code-block:: yaml

    my-ec2-config:
      # Specify whether to use public or private IP for deploy script
      # private_ips or public_ips
      ssh_interface: public_ips


Many EC2 instances do not allow remote access to the root user by default.
Instead, another user must be used to run the deploy script using sudo. Some
common usernames include ec2-user (for Amazon Linux), ubuntu (for Ubuntu
instances), admin (official Debian) and bitnami (for images provided by
Bitnami).

.. code-block:: yaml

    my-ec2-config:
      # Configure which user to use to run the deploy script
      ssh_username: ec2-user


Multiple usernames can be provided, in which case Salt Cloud will attempt to
guess the correct username. This is mostly useful in the main configuration
file:

.. code-block:: yaml

    my-ec2-config:
      ssh_username:
        - ec2-user
        - ubuntu
        - admin
        - bitnami


Multiple security groups can also be specified in the same fashion:

.. code-block:: yaml

    my-ec2-config:
      securitygroup:
        - default
        - extra


Block device mappings enable you to specify additional EBS volumes or instance
store volumes when the instance is launched. This setting is also available on
each cloud profile. Note that the number of instance stores varies by instance type.
If more mappings are provided than are supported by the instance type, mappings will be
created in the order provided and additional mappings will be ignored. Consult the
`AWS documentation`_ for a listing of the available instance stores, device names, and mount points.

.. code-block:: yaml

    my-ec2-config:
      block_device_mappings:
        - DeviceName: /dev/sdb
          VirtualName: ephemeral0
        - DeviceName: /dev/sdc
          VirtualName: ephemeral1

You can also use block device mappings to change the size of the root device at the 
provisioing time. For example, assuming the root device is '/dev/sda', you can set 
its size to 100G by using the following configuration.

.. code-block:: yaml

    my-ec2-config:
      block_device_mappings:
        - DeviceName: /dev/sda
          Ebs.VolumeSize: 100 


Tags can be set once an instance has been launched.

.. code-block:: yaml

    my-ec2-config:
        tag:
            tag0: value
            tag2: value

.. _`AWS documentation`: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html

Modify EC2 Tags
===============
One of the features of EC2 is the ability to tag resources. In fact, under the
hood, the names given to EC2 instances by salt-cloud are actually just stored
as a tag called Name. Salt Cloud has the ability to manage these tags:

.. code-block:: bash

    salt-cloud -a get_tags mymachine
    salt-cloud -a set_tags mymachine tag1=somestuff tag2='Other stuff'
    salt-cloud -a del_tags mymachine tag1,tag2,tag3


Rename EC2 Instances
====================
As mentioned above, EC2 instances are named via a tag. However, renaming an
instance by renaming its tag will cause the salt keys to mismatch. A rename
function exists which renames both the instance, and the salt keys.

.. code-block:: bash

    salt-cloud -a rename mymachine newname=yourmachine


EC2 Termination Protection
==========================
EC2 allows the user to enable and disable termination protection on a specific
instance. An instance with this protection enabled cannot be destroyed.

.. code-block:: bash

    salt-cloud -a enable_term_protect mymachine
    salt-cloud -a disable_term_protect mymachine


Rename on Destroy
=================
When instances on EC2 are destroyed, there will be a lag between the time that
the action is sent, and the time that Amazon cleans up the instance. During
this time, the instance still retails a Name tag, which will cause a collision
if the creation of an instance with the same name is attempted before the
cleanup occurs. In order to avoid such collisions, Salt Cloud can be configured
to rename instances when they are destroyed. The new name will look something
like:

.. code-block:: bash

    myinstance-DEL20f5b8ad4eb64ed88f2c428df80a1a0c


In order to enable this, add rename_on_destroy line to the main
configuration file:

.. code-block:: yaml

    my-ec2-config:
      rename_on_destroy: True


EC2 Images
==========
The following are lists of available AMI images, generally sorted by OS. These
lists are on 3rd-party websites, are not managed by Salt Stack in any way. They
are provided here as a reference for those who are interested, and contain no
warranty (express or implied) from anyone affiliated with Salt Stack. Most of
them have never been used, much less tested, by the Salt Stack team.

* `Arch Linux`__

  .. __: https://wiki.archlinux.org/index.php/Arch_Linux_AMIs_for_Amazon_Web_Services

* `FreeBSD`__

  .. __: http://www.daemonology.net/freebsd-on-ec2/

* `Fedora`__

  .. __: https://fedoraproject.org/wiki/Cloud_images

* `CentOS`__

  .. __: http://wiki.centos.org/Cloud/AWS

* `Ubuntu`__

  .. __: http://cloud-images.ubuntu.com/locator/ec2/

* `Debian`__

  .. __: http://wiki.debian.org/Cloud/AmazonEC2Image

* `Gentoo`__

  .. __: https://aws.amazon.com/amis?platform=Gentoo&selection=platform

* `OmniOS`__

  .. __: http://omnios.omniti.com/wiki.php/Installation#IntheCloud

* `All Images on Amazon`__

  .. __: https://aws.amazon.com/amis


show_image
==========
This is a function that describes an AMI on EC2. This will give insight as to
the defaults that will be applied to an instance using a particular AMI.

.. code-block:: bash

    $ salt-cloud -f show_image ec2 image=ami-fd20ad94


show_instance
=============
This action is a thin wrapper around --full-query, which displays details on a
single instance only. In an environment with several machines, this will save a
user from having to sort through all instance data, just to examine a single
instance.

.. code-block:: bash

    $ salt-cloud -a show_instance myinstance


del_root_vol_on_destroy
=======================
This argument overrides the default DeleteOnTermination setting in the AMI for
the EBS root volumes for an instance. Many AMIs contain 'false' as a default,
resulting in orphaned volumes in the EC2 account, which may unknowingly be
charged to the account. This setting can be added to the profile or map file
for an instance.

If set, this setting will apply to the root EBS volume

.. code-block:: yaml

    del_root_vol_on_destroy: True


This can also be set as a cloud provider setting in the EC2 cloud
configuration:

.. code-block:: yaml

    my-ec2-config:
      del_root_vol_on_destroy: True


del_all_vols_on_destroy
=======================
This argument overrides the default DeleteOnTermination setting in the AMI for
the not-root EBS volumes for an instance. Many AMIs contain 'false' as a
default, resulting in orphaned volumes in the EC2 account, which may
unknowingly be charged to the account. This setting can be added to the profile
or map file for an instance.

If set, this setting will apply to any (non-root) volumes that were created
by salt-cloud using the 'volumes' setting.

The volumes will not be deleted under the following conditions
* If a volume is detached before terminating the instance
* If a volume is created without this setting and attached to the instance

.. code-block:: yaml

    del_all_vols_on_destroy: True


This can also be set as a cloud provider setting in the EC2 cloud
configuration:

.. code-block:: yaml

    my-ec2-config:
      del_all_vols_on_destroy: True


The setting for this may be changed on all volumes of an existing instance
using one of the following commands:

.. code-block:: bash

    salt-cloud -a delvol_on_destroy myinstance
    salt-cloud -a keepvol_on_destroy myinstance
    salt-cloud -a show_delvol_on_destroy myinstance

The setting for this may be changed on a volume on an existing instance
using one of the following commands:

.. code-block:: bash

    salt-cloud -a delvol_on_destroy myinstance device=/dev/sda1
    salt-cloud -a delvol_on_destroy myinstance volume_id=vol-1a2b3c4d
    salt-cloud -a keepvol_on_destroy myinstance device=/dev/sda1
    salt-cloud -a keepvol_on_destroy myinstance volume_id=vol-1a2b3c4d
    salt-cloud -a show_delvol_on_destroy myinstance device=/dev/sda1
    salt-cloud -a show_delvol_on_destroy myinstance volume_id=vol-1a2b3c4d


EC2 Termination Protection
==========================

EC2 allows the user to enable and disable termination protection on a specific
instance. An instance with this protection enabled cannot be destroyed. The EC2
driver adds a show_term_protect action to the regular EC2 functionality.

.. code-block:: bash

    salt-cloud -a show_term_protect mymachine
    salt-cloud -a enable_term_protect mymachine
    salt-cloud -a disable_term_protect mymachine


Alternate Endpoint
==================
Normally, EC2 endpoints are build using the region and the service_url. The
resulting endpoint would follow this pattern:

.. code-block:: bash

    ec2.<region>.<service_url>


This results in an endpoint that looks like:

.. code-block:: bash

    ec2.us-east-1.amazonaws.com


There are other projects that support an EC2 compatibility layer, which this
scheme does not account for. This can be overridden by specifying the endpoint
directly in the main cloud configuration file:

.. code-block:: yaml

    my-ec2-config:
      endpoint: myendpoint.example.com:1138/services/Cloud


Volume Management
=================
The EC2 driver has several functions and actions for management of EBS volumes.


Creating Volumes
----------------
A volume may be created, independent of an instance. A zone must be specified.
A size or a snapshot may be specified (in GiB). If neither is given, a default
size of 10 GiB will be used. If a snapshot is given, the size of the snapshot
will be used.

.. code-block:: bash

    salt-cloud -f create_volume ec2 zone=us-east-1b
    salt-cloud -f create_volume ec2 zone=us-east-1b size=10
    salt-cloud -f create_volume ec2 zone=us-east-1b snapshot=snap12345678
    salt-cloud -f create_volume ec2 size=10 type=standard
    salt-cloud -f create_volume ec2 size=10 type=io1 iops=1000


Attaching Volumes
-----------------
Unattached volumes may be attached to an instance. The following values are
required; name or instance_id, volume_id and device.

.. code-block:: bash

    salt-cloud -a attach_volume myinstance volume_id=vol-12345 device=/dev/sdb1


Show a Volume
-------------
The details about an existing volume may be retrieved.

.. code-block:: bash

    salt-cloud -a show_volume myinstance volume_id=vol-12345
    salt-cloud -f show_volume ec2 volume_id=vol-12345


Detaching Volumes
-----------------
An existing volume may be detached from an instance.

.. code-block:: bash

    salt-cloud -a detach_volume myinstance volume_id=vol-12345


Deleting Volumes
----------------
A volume that is not attached to an instance may be deleted.

.. code-block:: bash

    salt-cloud -f delete_volume ec2 volume_id=vol-12345


Managing Key Pairs
==================
The EC2 driver has the ability to manage key pairs.


Creating a Key Pair
-------------------
A key pair is required in order to create an instance. When creating a key pair
with this function, the return data will contain a copy of the private key.
This private key is not stored by Amazon, and will not be obtainable past this
point, and should be stored immediately.

.. code-block:: bash

    salt-cloud -f create_keypair ec2 keyname=mykeypair


Show a Key Pair
---------------
This function will show the details related to a key pair, not including the
private key itself (which is not stored by Amazon).

.. code-block:: bash

    salt-cloud -f show_keypair ec2 keyname=mykeypair


Delete a Key Pair
-----------------
This function removes the key pair from Amazon.

.. code-block:: bash

    salt-cloud -f delete_keypair ec2 keyname=mykeypair


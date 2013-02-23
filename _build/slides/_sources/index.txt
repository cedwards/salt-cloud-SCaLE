============================================
Automated Cloud Provisioning with Salt Cloud
============================================

- Scale 11x
- Feb 22-24, 2013
- Christer Edwards

Disclaimer
----------

.. image:: /images/beta.jpg
   :align: center

What is Salt Cloud
------------------

Salt Cloud is a public cloud provisioning tool. It is meant to integrate into
cloud providers in a clean way so that systems on public clouds can be quickly
and easily modeled and provisioned.

Salt Cloud allows for cloud-based servers to be managed via virtual machine
maps and profiles. This means that cloud VMs can be created individually, or
large groups of systems can be managed at once.

What is Salt Cloud (cont.)
--------------------------

Virtual machines created with Salt Cloud install Salt on the target virtual
machine and assign it to the specified master. This means that virtual machines
can be provisioned and then potentially never logged into.

While Salt Cloud has been made to work with Salt, it is also a generic cloud
management platform and can be used to manage not Salt-centric clouds.

Who does Salt Cloud support?
----------------------------

.. image:: /images/cloud-providers.png

=============
Configuration
=============

/etc/salt/cloud
---------------

Core configuration options can be set in the cloud config file. By default this
file is located at ``/etc/salt/cloud``.

The default minion configuration is set up in this file:

.. code-block:: yaml

    minion:
        master: salt.domain.tld

Cloud Configurations
--------------------

Data specific to interacting with public clouds is set up here as well.

**Amazon AWS**

.. code-block:: yaml

    AWS.id:
    AWS.key:
    AWS.keyname:
    AWS.securitygroup:
    AWS.private_key:

**Linode**

.. code-block:: yaml

    LINODE.apikey:
    LINODE.password:

Cloud Configurations (cont.)
----------------------------

**Joyent Cloud**

.. code-block:: yaml

    JOYENT.user:
    JOYENT.password:
    JOYENT.private_key:

**GoGrid**

.. code-block:: yaml

    GOGRID.apikey:
    GOGRID.sharedsecret:

Cloud Configurations (cont.)
----------------------------

**IBM Smart Cloud Enterprise**

.. code-block:: yaml

    IBMSCE.user:
    IBMSCE.password:
    IBMSCE.ssh-key_name:
    IBMSCE.ssh_key_file:
    IBMSCE.location:

**OpenStack for HP**

.. code-block:: yaml

    OPENSTACK.identity_url:
    OPENSTACK.compute_name:
    OPENSTACK.compute_region:
    OPENSTACK.tenant:
    OPENSTACK.user:
    OPENSTACK.ssh_key_name:
    OPENSTACK.ssh_key_file:
    OPENSTACK.password:

Cloud Configurations (cont.)
----------------------------

**OpenStack for RackSpace**

.. code-block:: yaml

    OPENSTACK.identity_url:
    OPENSTACK.compute_name:
    OPENSTACK.compute_region:
    OPENSTACK.tenant:
    OPENSTACK.user:
    OPENSTACK.password:
    OPENSTACK.protocol:

/etc/salt/cloud
---------------

**Example**

.. code-block:: yaml

    AWS.id: 'HJGRYCILJLKJYG'
    AWS.key: 'kdjgfsgm;woormgl/aserigjksjdhasdfgn'
    AWS.keyname: test
    AWS.securitygroup: quick-start
    AWS.private_key: /root/test.pem

    minion:
      master: salt.domain.tld

===========
VM Profiles
===========

Profiles
--------

Salt Cloud designates virtual machines inside the profile configuration file.
This file defaults to ``/etc/salt/cloud.profiles`` and is a yaml configuration. 

.. code-block:: yaml

    fedora_aws:
      provider: aws
      image: ami-6145cc08
      size: Micro Instance
      ssh_interface: public
      ssh_username: ec2-user

    centos_linode:
      provider: linode
      image: CentOS 6.2 64bit
      size: Linode 512

.. code-block:: bash

    [root@master ~]# salt-cloud -p centos_linode web1 web2

VM Profiles (cont.)
-------------------

It's possible to define the Salt master as well as custom grains in the profile
itself. This allows you to assign roles as well as point to different Salt
masters within different providers (read: syndics).

**note**: Automated syndic key management is incomplete yet.

.. code-block:: yaml

    cent_linode:
      provider: linode
      image: 'CentOS 6.2 64bit'
      size: 'Linode 512'
      minion:
        master: salt.domain.tld
      grains:
        role: webserver

====
Maps
====

Cloud Map File
--------------

Map files have a simple format: Specify a profile and then a list of virtual
machines to make from said profile.

.. code-block:: yaml

    fedora_aws:
      - web1
      - web2
      - web3
    fedora_high:
      - redis1
      - redis2
      - redis3
    cent_high:
      - mysql1
      - mysql2
      - mysql3

.. code-block:: bash

    [root@master ~]$ salt-cloud -m /path/to/map.file
    [root@master ~]$ salt-cloud -m /path/to/map.file -P

Cloud Map File (cont.)
----------------------

Cloud map files can also include grains:

.. code-block:: yaml

    fedora_small:
      - web1:
          grains:
            role: webserver
      - web2:
          grains:
            role: mysql
      ...

==================
Bootstrapping Salt
==================

Salt Cloud supports a few methods for bootstrapping Salt onto the newly
provisioned machine.

 - script: (<0.8.4)
 - salt-bootstrap (>=0.8.4)

script:
-------

``site-packages/saltcloud/deploy/`` includes a number of distribution-specific
installation scripts. Example: ``Fedora.sh``

.. code-block:: bash

    #!/bin/bash
    yum install -y salt-minion
    # Save the minion public and private RSA keys
    mkdir -p /etc/salt/pki
    echo '{{ vm['priv_key'] }}' > /etc/salt/pki/minion.pem
    echo '{{ vm['pub_key'] }}' > /etc/salt/pki/minion.pub
    # Copy the minion configuration file into place
    echo '{{ minion }}' > /etc/salt/minion
    # Set the minion to start on reboot
    systemctl enable salt-minion.service
    # Start the minion!
    systemctl start salt-minion.service

script: usage
---------------

To use a distribution-specific deployment script, simply use the ``script:``
option within the profile definition.

.. code-block:: yaml
   :emphasize-lines: 5

    centos_linode:
      provider: linode
      image: CentOS 6.2 64bit
      size: Linode 512
      script: RHEL6

salt-bootstrap
--------------

This script runs through a series of checks to determine operating system type
and version to then install the Salt binaries using the appropriate methods.

.. code-block:: yaml
   :emphasize-lines: 5

    centos_linode:
      provider: linode
      image: CentOS 6.2 64bit
      size: Linode 512
      script: bootstrap-salt

**note**: salt-bootstrap is the default as of ``0.8.4``. If you omit the script
option, salt-bootstrap will be used.

.. code-block:: bash

    [root@master ~]# salt-cloud [-u|--update-bootstrap]

Setting up Salt Masters
-----------------------

.. code-block:: yaml
   :emphasize-lines: 6-9

    centos_linode:
      provider: linode
      image: CentOS 6.2 64bit
      size: Linode 512
      script: bootstrap-salt
      make_master: True
      master:
        user: root
        interface: 0.0.0.0

No deployment
-------------

It is also possible to simply provision instances without Salting them.

.. code-block:: yaml

    deploy: False
    script: None

.. code-block:: bash

    [root@master ~]# salt-cloud --no-deploy -p profile instance

====
Demo
====

==============
What's Coming?
==============

Salt 0.14.0 will include initial versions of Salt-Virt

- KVM support first (libvirt)
- ``salt.run virt.query``
- ``salt.run virt.init $name $cpu $ram $image``

 - Images served from: salt://, http://, ftp://

==========
Questions?
==========

=======
Credits
=======

- Christer Edwards (christer.edwards@gmail.com)
- Salt Stack - http://saltstack.com
- Salt Cloud Docs - http://salt-cloud.readthedocs.org
- Into the Salt Mine - http://intothesaltmine.org

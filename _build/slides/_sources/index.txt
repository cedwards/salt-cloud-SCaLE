============================================
Automated Cloud Provisioning with Salt Cloud
============================================

- Scale 11x
- Feb 22-24, 2013
- Christer Edwards

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

/etc/salt/cloud
---------------

Core configuration options can be set in the cloud config file. By default this
file is located at ``/etc/salt/cloud``.

The default minion configuration is set up in this file:

.. code-block:: yaml

    minion:
        master: master.domain.tld

Cloud Configurations
--------------------

Data specific to interacting with public clouds is set up here as well.

**Rackspace**

.. code-block:: yaml

    RACKSPACE.user:
    RACKSPACE.apikey:

**Amazon AWS**

.. code-block:: yaml

    AWS.id:
    AWS.key:
    AWS.keyname:
    AWS.securitygroup:
    AWS.private_key:

Cloud Configurations (cont.)
----------------------------

**Linode**

.. code-block:: yaml

    LINODE.apikey:
    LINODE.password:

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

    AWS.id: HJGRYCILJLKJYG
    AWS.key: 'kdjgfsgm;woormgl/aserigjksjdhasdfgn'
    AWS.keyname: test
    AWS.securitygroup: quick-start
    AWS.private_key: /root/test.pem

    minion:
      master: saltmaster.example.com

VM Profiles
-----------

Salt Cloud designates virtual machines inside the profile configuration file. The profile configuration file defaults to ``/etc/salt/cloud.profiles`` and is a yaml configuration. The syntax for declaring profiles is simple:

**Example**

.. code-block:: yaml

    cent_linode:
      provider: linode
      image: CentOS 6.2 64bit
      size: Linode 512
      script: RHEL6

    cent_gogrid:
      provider: gogrid
      image: 12834
      size: 512MB
      script: RHEL6

Cloud Map File
--------------

Map files have a simple format: Specify a profile and then a list of virtual machines to make from said profile.

.. code-block:: yaml

    fedora_small:
        - web1
        - web2
        - web3
    fedora_high:
        - redis1
        - redis2
        - redis3
    cent_high:
        - riak1
        - riak2
        - riak3

====
Demo
====

==========
Questions?
==========

=======
Credits
=======

- Christer Edwards (christer.edwards@gmail.com)
- Salt Stack - http://saltstack.com
- Into the Salt Mine - http://intothesaltmine.org

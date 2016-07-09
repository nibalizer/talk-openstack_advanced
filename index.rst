
.. Secure Peer Networking with TINC slides file, created by
   hieroglyph-quickstart on Sun Nov 15 21:40:13 2015.


=========================
OpenStack Advanced Topics
=========================

.. figure:: _static/TuxTotem.png
   :align: left
   :width: 400px

Spencer Krum, IBM

April 24th, 2016

@nibalizer

.. note::

   * Who am I
   * What do I work on
   * github
   * This talk is my version of an introduction to openstack
   * demystify
   * objective truths
   * no vendor pitch
   * i want you to know the words when openstack is being discussed
   * I'll also show you how to use it


Portland
========

.. figure:: _static/mt_hood.jpg
   :align: center




.. slide:: 
   :level: 2

   .. figure:: _static/openstack-cloud-software-vertical-large.png
      :align: center


.. slide:: 
   :level: 2

   .. figure:: _static/oh-snap-gif-1.gif
      :align: center

Agenda
======

* Introduction
* Basic OpenStack Stuff
* Manilla
* Designate
* Barbican
* Magnum
* lbaas
* DevStack
* OpenStack Debugging
* Ironic/Bifrost



Who Am I?
=========


.. note::
    * Sysop
    * PSU
    * Work at IBM
    * I build the infrastructure that tests openstack, jenkins, code hosting
    * Access to a good half dozen clouds
    * boot nodes all the time
    * Working at IBM to deploy a cloud

OpenStack Mission Statement
===========================

.. code-block:: shell

    To produce a ubiquitous Open Source Cloud Computing platform that is
    easy to use, simple to implement, interoperable between deployments,
    works well at all scales, and meets the needs of users and operators of
    both public and private clouds.


Basic Things you can ask an OpenStack to do
===========================================

* Make a vm
* Destroy a vm
* Set a dns record
* Store a file
* Create a 2T disk and attach it to a vm
* Create a mysql database


DevStack
========


People say to me: "Spencer, i tried to install openstack and i couldn't"

Enter DevStack




Less Basic things you can ask an OpenStack to do
================================================

* Snapshot a vm
* Upload an image to boot new vms from
* Create an L2 network segment that several vms are all tapped into
* Create a Load Balancer and add groups to it
* Setup rules for scaling up and down automatically
* Host Containers, or container orchestration engines
* Create, attach, and move floating ips


What is OpenStack
=================


* Python Daemons
* Infrastructure as a Service
* Open Source


.. note::
    * Python daemon that takes in rest api and then causes other things to happen
    * Some kind of a programmable thing that does stuff that datacenter techs used to do
    * Tickets!
    * Ticket to get a vm
    * Rest API to get a vm
    * Apache 2




What OpenStack is Not
=====================


* Hypervisor
* Amazon


.. note::
    * Xen, Kvm, Virtualbox, Vmware these are hypervisors
    * Amazon web services, its not that and its not compatible
    * Eucalyptus


Definitions
===========

* User
* Operator
* Network
* Subnet
* Hypervisor
* Compute host
* Controller
* Instance
* Cloud

.. note::
    * a subnet is l3
    * a network is l2


The Four Opens
==============

* Open Source
* Open Design
* Open Development
* Open Community


.. note::
    * Not Open Core, Apache2
    * Design is open and open to contributors
    * The development is done in the open with open tooling
    * The discussion and voting and technical direction is all transparent
    * There is a CoC



History
=======

* Started 2010
* Collaboration between Rackspace and NASA
* Releases every 6 months
* Mitaka is out
* Newton design summit starts tomorrow

.. note::
    * I started working on it in 2014


Fast Facts
==========

* ~600 git repos
* ~7k emails / 6 mo
* 20k commits / 6 mo
* 100k reviews / 6 mo

.. note::
    * openstack development is freaking huge


Primary Services
================

* Nova
* Neutron
* Glance
* Cinder
* Keystone
* Swift
* Trove
* Designate



Iaas UX
=======

* Invisible/No Interaction
* Web UI
* Command line utility
* Deployment Tool
* Library

.. note::
    * OpenStack has a UX Team
    * What is cloud ux



.. slide:: 
   :level: 2

   .. figure:: _static/horizon_1.png
      :align: center

.. slide:: 
   :level: 2

   .. figure:: _static/horizon_2.png
      :align: center


CLI: Env Vars
=============

.. figure:: _static/env-vars.gif
   :align: center


CLI: List Machines
==================

.. figure:: _static/nova-list.gif
   :align: center


CLI: Show Machine
=================

.. figure:: _static/nova-show.gif
   :align: center


CLI: Create Machine
===================

.. figure:: _static/nova-boot.gif
   :align: center


CLI: Destroy Machine
====================

.. figure:: _static/nova-delete.gif
   :align: center

CLI: Future
===========

.. figure:: _static/openstack-server-list.gif
   :align: center

CLI: Recap
=============

Upload a new image

.. code-block:: shell

   nova list
   nova boot
   openstack server list
   openstack server create
   openstack flavor list
   openstack image list


CLI: Advanced
=============

Upload a new image

.. code-block:: shell

    openstack image create --disk-format qcow2 \
    --container-format bare --file mynixosimg.qcow nixos

CLI: Advanced
=============

Upload a file to swift

.. code-block:: shell

    openstack conatiner create test1
    openstack object create test1 mypicture.png


Deployment: List Hosts
======================

.. code-block:: shell

    $ ansible all -i openstack.py  --list-hosts
      hosts (1):
        cacti-hodor-dfc7a021-3d50-4c3c-8082-a0aecb6d3878


Deployment: Playbook
====================

.. code-block:: yaml

    ---
      - name: Foo
        hosts: localhost
        connection: local
        vars:
          FLAVOR: '8GB Standard Instance' 
          IMAGE_NAME: 'Ubuntu 14.04 LTS (Trusty Tahr) (PVHVM)'
          KEY_NAME: nibz
        tasks:
          - name: create instances
            os_server:
              name: "{{ item }}"
              image: "{{ IMAGE_NAME }}"
              key_name: "{{ KEY_NAME }}"
              wait: yes
              timeout: 200
              flavor: "{{ FLAVOR }}"
            with_items:
              - foo
              - bar
              - baz


Deployment: Boot Many Machines
==============================

.. code-block:: shell

    $: ansible-playbook -i openstack.py ansible_machines.yml 

    PLAY [Foo] *********************************************************************

    TASK [setup] *******************************************************************
    ok: [localhost]

    TASK [create instances] ********************************************************
    changed: [localhost] => (item=foo)
    changed: [localhost] => (item=bar)
    changed: [localhost] => (item=baz)

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0   


Deployment: Results
==============================

.. code-block:: shell

    $: ansible all -i openstack.py --list-hosts
      hosts (5):
        twitch-hodor-4b73cb8d-d2b2-4dc6-a533-486d816e45f1
        bar
        foo
        baz
        cacti-hodor-dfc7a021-3d50-4c3c-8082-a0aecb6d3878



Library: Shade
==============

* Technically you can import the python-novaclient library directly
* Generally you don't want to do that
* Shade wraps all the libraries with a common model
* Nice things like rate limiting, exception handling


Library: OpenStack Client Config
================================

.. code-block:: yaml

    clouds:
      mordred:
        profile: hp
        auth:
          username: mordred@inaugust.com
          password: XXXXXXXXX
          project_name: mordred@inaugust.com
        region_name: region-b.geo-1
        dns_service_type: hpext:dns
        compute_api_version: 1.1
      monty:
        auth:
          auth_url: https://region-b.geo-1.identity.hpcloudsvc.com:35357/v2.0
          username: monty.taylor@hp.com
          password: XXXXXXXX
          project_name: monty.taylor@hp.com-default-tenant
        region_name: region-b.geo-1
        dns_service_type: hpext:dns


Library: Shade usage
====================

.. code-block:: python

    cloudname = sys.argv[1]
    cloud = shade.openstack_cloud(name=cloudname)
    image = filter_images('trusty', cloud.list_images())
    server_name = human_name + "-hodor-" + str(uuid.uuid4())

    cloud.create_server(server_name, image['id'], flavor['id'], key_name=key[0]['id'])


Further Services
================

* Manilla
* Designate
* Barbican
* Magnum
* lbaas
* Ironic/Bifrost

Manilla
=======


* "shared file system service for OpenStack"
* did not work for me


Barbican
========

* Barbican is a REST API designed for the secure storage, provisioning and management of secrets such as passwords, encryption keys and X.509 Certificates.
* Let's Encrypt makes this less of a requirement
* Interesting relationship with keystone
* Presentations going back to grizzly



Desginate
=========


* DNS as a service
* Proxy
* Vexxhost has this deployed
* hpcloud used to :(


Desginate
=========


.. code-block:: shell

    DNS_DOMAIN_ID=9609dad3-fc98-451f-9bfc-0978be5733c5

    designate --os-endpoint  \
    https://region-a.geo-1.dns.hpcloudsvc.com/v1/  \
    record-list 9609dad3-fc98-451f-9bfc-0978be5733c5


Magnum
======

* Container Orchestration Engine
* Built on Heat
* Bays


Magnum
======


.. code-block:: shell

    magnum bay-create --name k8s_bay --baymodel kubernetes --node-count 2



Database Investigation
======================

.. code-block:: shell

    $: nova hypervisor-list
    +----+---------------------+-------+---------+
    | ID | Hypervisor hostname | State | Status  |
    +----+---------------------+-------+---------+
    | 1  | osat-00             | up    | enabled |
    +----+---------------------+-------+---------+

    mysql> select id,created_at,host,uuid from compute_nodes;
    +----+---------------------+---------+--------------------------------------+
    | id | created_at          | host    | uuid                                 |
    +----+---------------------+---------+--------------------------------------+
    |  1 | 2016-04-21 22:57:02 | osat-00 | 90dd1911-55da-417b-955f-8412b6405043 |
    +----+---------------------+---------+--------------------------------------+

Database Investigation
======================

.. code-block:: shell

    $: openstack server list
    +--------------------------------------+------+--------+------------------+
    | ID                                   | Name | Status | Networks         |
    +--------------------------------------+------+--------+------------------+
    | 4b9c6d10-adbe-476a-ada8-fd74b4ba14f5 | derp | ACTIVE | private=10.0.0.2 |
    +--------------------------------------+------+--------+------------------+


    mysql> select created_at,id,display_name,node from instances;
    +---------------------+----+--------------+---------+
    | created_at          | id | display_name | node    |
    +---------------------+----+--------------+---------+
    | 2016-04-21 23:25:47 |  1 | derp         | osat-00 |
    +---------------------+----+--------------+---------+


Rabbit Poking
=============


.. code-block:: shell

    root@osat-00:~# rabbitmqctl list_queues

    Listing queues ...
    cinder-scheduler0
    cinder-scheduler.osat-000
    cinder-scheduler_fanout_e047d94558b54154814054311e76417e0
    cinder-volume0
    cinder-volume.osat-00@lvmdriver-100
    cinder-volume_fanout_756d498d0c9f4f409c2db54f71e6f19e0
    compute_nodes0
    compute.osat-000
    compute_fanout_2369c871bbd745ed993afaf8311926770
    conductor0
    conductor.osat-0000
    conductor_fanout_442efef3b5a5471bbb91f36fe18afbea0
    conductor_fanout_97a06467f07d4a26a3dccf5112a94fb30
    consoleauth0
    consoleauth.osat-00000
    consoleauth_fanout_2b3e4c3f17fa424caef5b1ffc2ae5aba0
    Networks0
    network.osat-000000
    network_fanout_4c3fcd09064e4503ba717080f7bd45ff0
    reply_48397d82474d45568675f3b68b054f840
    reply_9e1f6ca491e74195a12b8acdaa6bea030
    reply_cfecefe6614b4a299a1c38d41b57e9170
    reply_ff1ca09e486349198cbe497df90b00b30
    scheduler_fanout_e047d94558b54154814054311e76417e00
    scheduler.osat-0000000
    scheduler_fanout_2c5743eaed1a4a608794b19bfecce42e0
    ...done.

Grab a specific queue
=====================


.. code-block:: shell


    ./rabbitmqadmin get queue=compute.osat-00 > file

    ...  "task_state": "deleting", "shutdown_terminate": false, ...
    "task_state": "deleting"

.. note::

    * i had to stop nova-compute in order to get the queue to actually fill up enough to look at an element in it
    * The blob you get looks like a database entry.then there is escaped json containing more escaped json
    * start and stopping nova-compute causes things to queue then they start again when nova compute runs again



SoapBox
=======


Flavors, images, networks, regions

If you want to boot a vm you need to know all 4 of these things
The problem is every cloud is different

Consistent flavor names and image names, and network names would make it easier
Standards things

image std-ubuntu-14.04
flavor std-small-1





References
==========

* OpenStack Foundation Website: http://www.openstack.org/
* DevStack: http://docs.openstack.org/developer/devstack/overview.html
* OS Maintenance: http://docs.openstack.org/openstack-ops/content/maintenance.html (Look under: Total compute node failure)
* DB Surgery: https://thornelabs.net/2014/08/03/delete-duplicate-openstack-hypervisors-and-services.html


References (cont)
=================

* Rabbitmqadmin http://www.rabbitmq.com/management.html
* oslo.messaging https://wiki.openstack.org/wiki/Oslo/Messaging#Concepts


References
==========

* https://wiki.openstack.org/wiki/Magnum
* https://wiki.openstack.org/wiki/Designate
* https://wiki.openstack.org/wiki/Manila
* https://wiki.openstack.org/wiki/Barbican



Thank You + Questions
=====================

.. figure:: _static/spencer_face.jpg
   :align: left

Spencer Krum

IBM

@nibalizer

nibz@spencerkrum.com

https://github.com/nibalizer/talk-openstack_for_humans


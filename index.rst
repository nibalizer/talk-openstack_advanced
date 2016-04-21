
.. Secure Peer Networking with TINC slides file, created by
   hieroglyph-quickstart on Sun Nov 15 21:40:13 2015.


====================
OpenStack For Humans
====================

.. figure:: _static/cascadia_logo.png
   :align: left
   :width: 400px

Spencer Krum, IBM

March 12, 2016

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
* OpenStack Word Definitions
* OpenStack UX
* Examples


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
* Mitaka is comming out RSN

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


References
==========

* OpenStack Foundation Website: http://www.openstack.org/
* Ansible shade: https://github.com/ansible/ansible-modules-core/tree/devel/cloud/openstack
* Shade: http://git.openstack.org/cgit/openstack-infra/shade
* Tim Chavez on Ansible: http://busywait.com/using-ansible-to-concurrently-boot-openstack-vms/
* OS Client Config: http://docs.openstack.org/developer/os-client-config/
* Hodorv2: https://github.com/nibalizer/hodor
* Hodorv1: https://github.com/nibalizer/dotfiles/blob/master/local/bin/hodor



References (cont)
=================

* Nova Client http://docs.openstack.org/developer/python-novaclient/api.html
* OpenStack Client: http://docs.openstack.org/developer/python-openstackclient/
* OpenStack UX Team: https://wiki.openstack.org/wiki/UX
* Getting started contributing https://wiki.openstack.org/wiki/How_To_Contribute
* Ansible openstack inventory: https://github.com/ansible/ansible/blob/devel/contrib/inventory/openstack.py
* Terraform OpenStack provider: https://www.terraform.io/docs/providers/openstack/index.html



Thank You + Questions
=====================

.. figure:: _static/spencer_face.jpg
   :align: left

Spencer Krum

IBM

@nibalizer

nibz@spencerkrum.com

https://github.com/nibalizer/talk-openstack_for_humans


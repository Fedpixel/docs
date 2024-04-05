Cisco
=====

BGP
---

Assigning an IP to an interface

.. code-block:: Text

   interface FastEthernet0/0
   ip address 192.168.1.1 255.255.255.0
   no shutdown

Defining an Autonomous System (AS)

.. code-block:: Text

   router bgp 4200000001
   neighbor 172.16.0.2 remote-as 4200000002

Displaying routes

.. code-block:: Text

   show ip route

Displaying the current configuration

.. code-block:: Text

   show running-config

Displaying BGP configuration

.. code-block:: Text

   show ip bgp summary

Disabling logs in the terminal

.. code-block:: Text

   no logging console

Adding a static route

.. code-block:: Text

   ip route 192.168.1.0 255.255.255.0 172.16.0.1

Distributing connected routes

.. code-block:: Text

   redistribute connected

One router to rule them all

.. code-block:: Text

   ip route 0.0.0.0 0.0.0.0 null 0
   router bgp 4200000301
   network 0.0.0.0 mask 0.0.0.0
   neighbor 172.16.0.1 next-hop-self
   neighbor 172.16.0.2 next-hop-self

Distributing bgp routes

.. code-block:: Text

   redistribute bgp

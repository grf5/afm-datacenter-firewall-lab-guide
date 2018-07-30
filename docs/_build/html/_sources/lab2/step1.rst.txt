Lab Overview
============

New in the v13 release of the BIG-IP Advanced Firewall Manager is the capability to insert a packet trace into the internal flow so you can analyze what component within the system is allowing or blocking packets based on your configuration of features and rule sets.

|image41|

The packet tracing is inserted at L3 immediately prior to the Global IP
intelligence. Because it is after the L2 section, this means that:

a) we cannot capture in tcpdump so we can’t see them in flight and

b) no physical layer details will matter as it relates to testing.

That said, it’s incredibly useful for what is and is not allowing your
packets through. You can insert tcp, udp, sctp, and icmp packets, with a
limited set of (appropriate to each protocol) attributes for each.

Lab 1 – Advanced Firewall Manager (AFM) Packet Tracer
=====================================================

.. _section-2:

Create and View Packet Tracer Entries
-------------------------------------

In this section, you will generate various types of traffic as you did
previously, but now you will view the flow using the network packet
tracer. Login to bigip2.dnstest.lab

(192.168.1.150), navigate to **Security > Debug > Packet Tester**.

|image42|

Create a packet test with the following parameters:

+-------------------+------------------------+
| **Protocol**      | TCP                    |
+===================+========================+
| **TCP Flags**     | SYN                    |
+-------------------+------------------------+
| **Source**        | IP - 1.2.3.4           |
|                   | Port – 9999            |
|                   | Vlan – Outside         |
+-------------------+------------------------+
| **TTL**           | 255                    |
+-------------------+------------------------+
| **Destination**   | IP – 10.30.0.50        |
|                   | Port - 80              |
+-------------------+------------------------+
| **Trace Options** | Use Staged Policy – no |
|                   | Trigger Log - no       |
+-------------------+------------------------+

Click Run Trace to view the response. Your output should resmeble the
allowed flow as shown below:

|image43|

You can also click on the “Route Domain Rules” trace result and see
which rule is permitting the traffic.

|image44|

Click **New Packet Trace** (optionally do not clear the existing data –
aka leave checked).

Create a packet test with the following parameters:

+-------------------+------------------------+
| **Protocol**      | TCP                    |
+===================+========================+
| **TCP Flags**     | SYN                    |
+-------------------+------------------------+
| **Source**        | IP - 1.2.3.4           |
|                   | Port – 9999            |
|                   | Vlan – Outside         |
+-------------------+------------------------+
| **TTL**           | 255                    |
+-------------------+------------------------+
| **Destination**   | IP – 10.30.0.50        |
|                   | Port - **8081**        |
+-------------------+------------------------+
| **Trace Options** | Use Staged Policy – no |
|                   | Trigger Log - no       |
+-------------------+------------------------+

Click Run Trace to view the response. Your output should resmeble the
allowed flow as shown below:

|image45|

This shows there is no rule associated with the route domain or a
virtual server which would permit the traffic. As such, the traffic
would be dropped/rejected.

Lab 2 – Advanced Firewall Manager (AFM) Flow Inspector
======================================================

Create and View Flow Inspector Data
-----------------------------------

A new tool introduced in version 13 is the flow inspector. This tool is
useful to view statistical information about existing flows within the
flow table. To test the flow inspector, navigate to **Security > Debug >
Flow Inspector.** Refresh the web page we’ve been using for testing
(http://10.30.0.50) and click “Get Flows”.

|image46|

Select a flow and click on the pop-out arrow

for additional data.

|image47|

This will show the TMM this is tied to as well as the last hop and the
idle timeout. This data is extremely valuable when troubleshooting
application flows.

It is also worth noting you can click directly on the IP address of a
flow to pre-populate the data in the packet tester for validating access
and/or where the flow is permitted.

Lab 3 – Stale Rule Report
=========================

AFM also can list out stale rules within the device its self. You must
first enable the feature. To enable, navigate to **Security >Reporting >
Settings > Reporting Settings.** You will then need to check
“\ **Collect Stale Rules Statistics**\ ” found under the Network
Firewall Rules Section. Please be sure to click “Save” before
proceeding.

|image48|

Once enabled, navigate to **Security >Reporting > Network > Stale
Rules.** Feel free to refresh the web page we’ve been testing with
(http://10.30.0.50) to see data populate into the rules.

**Note: this could take >60 seconds to populate the data**

|image49|

This information is quite useful for keeping a rule base tidy and
optimized.

**Anyone can create a firewall rule, but who is the person that removes
the unneccesary ones?**

   |image50|

.. |image40| image:: ../media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |image41| image:: ../media/image40.png
   :width: 6.5in
   :height: 3.44792in
.. |image42| image:: ../media/image41.png
   :width: 6.48958in
   :height: 3.44792in
.. |image43| image:: ../media/image42.png
   :width: 6.47361in
   :height: 2.94722in
.. |image44| image:: ../media/image43.png
   :width: 6.5in
   :height: 2.66667in
.. |image45| image:: ../media/image44.png
   :width: 6.49722in
   :height: 2.97708in
.. |image46| image:: ../media/image45.png
   :width: 6.48542in
   :height: 1.34653in
.. |image47| image:: ../media/image46.png
   :width: 6.49167in
   :height: 0.68819in
.. |image48| image:: ../media/image47.png
   :width: 3.55556in
   :height: 3.70347in
.. |image49| image:: ../media/image48.png
   :width: 6.49722in
   :height: 1in
.. |image50| image:: ../media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in

|image0|

**F5 BIG-IP Training**

**Advanced Firewall Manager**

**(AFM) – Lab01**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

|https://www.icsalabs.com/sites/default/files/imagecache/large_logo/ICSA_Cert_Firewall_WEB.gif|

Connecting to the Training Lab
==============================

The training lab is accessed over remote desktop connection.

Your administrator will provide login credentials and the URL/FQDN

Within each lab environment there are the following Virtual Machines:

-  **Windows 7 Jumpbox**

   -  Jumpbox – 192.168.1.200

-  **Two BIG-IP Virtual Editions (VE) – TMOS 13.1.0.1**

   -  BIG-IP 13.1.0.1 – DNS/DOS – 192.168.1.100

   -  BIG-IP 13.1.0.1 – Firewall – 192.168.1.150

-  **Two BIG-IQ Virtual Editions (VE) – TMOS 6.0**

   -  BIG-IQ 6.0 – Central Management (CM) – 192.168.1.50

   -  BIG-IQ 6.0 – Data Collection Device (DCD) – 192.168.1.60

-  **Four Ubuntu 17.10 LTS Hosts**

   -  Victim Server – 10.10.0.50

   -  Attack Host – 10.20.0.50

   -  Web Server – 10.30.0.50

   -  Application Server – 10.40.0.50

|image2|

.. _section-1:

Ravello Environment pre-check
=============================

When you RDP in to the Windows box, make sure the time on the windows
client is correct for the current time in the Eastern Time Zone.

If the time needs to be corrected, click on the clock and choose “Change
date and time settings…”

|image3|

**If the time is not corrected, please alert an instructor before
proceeding**.

Lab Overview
============

During this lab, you will configure the BIG-IP system to permit traffic
to multiple backend servers. You will then run simulated user flows
against BIG-IP and verify the traffic flow, reporting and logging of
these flows.

Base BIG-IP Configuration
-------------------------

In this lab, the VE has been configured with the basic system settings
and the VLAN/self-IP configurations required for the BIG-IP to
communicate and pass traffic on the network. We’ll now need to configure
the BIG-IP to pass it to the back-end server.

Lab 1 – Advanced Firewall Manager (AFM)
=======================================

Welcome to Initech! Today is your first day as the principal firewall
engineer, congratulations! The employee you are replacing, Milton, is
rumored to be sitting on a beach in Key West sipping Mai Tai’s and took
his red stapler but left no documentation…

The marketing team, now led by Bill Lumbergh, launched a new campaign
for Initiech’s TPS reports overnight and no one can access the web
server. The only information the web server administrators know is that
the IP address of the Web server is 10.30.0.50 and that Mr. Lumbergh is
furious the world does not know about the glory of TPS reports!!

Let’s start by testing the web server to verify. On your workstation
open a browser (we prefer you use the Chrome shortcut labeled BIG-IP UI,
all the tabs are pre-populated) and enter the address of the web server
(http://10.30.0.50). No Bueno! Let’s see if we can even ping the host.
Launch a command prompt (startrun cmd) and type ‘ping 10.30.0.50’.
Bueno! Looks like the server is up and responding to pings, as such,
this is likely not a network connectivity issue.

You ask one of your colleagues, who just got out of his meeting with the
Bob’s, if he knows the IP address of the firewall. He recalls the
firewall they would traverse for this communication is
bigip2.dnstest.lab and its management IP address is 192.168.1.150. In
your browser, open a new tab (of if you’re using Chrome open the tab
with bigip2.dnslab.lab) and navigate to https://192.168.1.150. The
credentials to log into the device are username: admin and password:
401elliottW! (these can also be found on the login banner of the device
for convenience). Note if you receive a security warning it is ok to
proceed to the site and add as a trusted site.

F5? F5 makes a data center firewall? Maybe I should do a little reading
about what the F5 firewall is before I proceed deeper into the lab…

Advanced Firewall Manager (AFM)
===============================

Advanced Firewall Manager (AFM) is a module that was added to TMOS in
version 11.3. F5

BIG-IP Advanced Firewall Manager™ (AFM) is a high-performance ICSA
certified, stateful, full-proxy

network firewall designed to guard data centers against incoming threats
that enter the network on the most widely deployed protocols—including
HTTP/S, SMTP, DNS, SIP, and FTP.

By aligning firewall policies with the applications, they protect,
BIG-IP AFM streamlines application deployment, security, and monitoring.
With its scalability, security and simplicity, BIG-IP AFM forms the core
of the F5 application delivery firewall solution.

|image4|

Some facts below about AFM and its functionality:

-  Advanced Firewall Manager (AFM) provides “Shallow” packet inspection
   while Application Security Manager (ASM) provides “Deep” packet
   inspection. By this we mean that AFM is concerned with source IP
   address and port, destination IP address and port, and protocol (this
   is also known as 5-tuple/quintuple filtering).

-  AFM is used to allow/deny a connection before deep packet inspection
   ever takes place, think of it as the first line of firewall defense.

-  AFM is many firewalls in one. You can apply L4 firewall rules to ALL
   addresses on the BIG-IP or you can specify BIG-IP configuration
   objects (route domains, virtual server, self-IP, and Management-IP).

-  AFM runs in 2 modes: **ADC mode** and **Firewall** mode. **ADC mode**
   is called a “blacklist”, all traffic is allowed to BIG-IP except
   traffic that is explicitly DENIED (this is a negative security
   model). **Firewall mode** is called a “whitelist”, all traffic is
   denied to BIG-IP except traffic that is explicitly ALLOWED. The
   latter is typically used when the customer only wants to use us as a
   firewall or with LTM.

-  We are enabling “SERVICE DEFENSE IN DEPTH” versus traditional
   “DEFENSE IN DEPTH”. This means, instead of using multiple shallow and
   deep packet inspection devices inline increasing infrastructure
   complexity and latency, we are offering these capabilities on a
   single platform.

-  AFM is an ACL based firewall. In the old days, we used to firewall
   networks using simple packet filters. With a packet filter, if a
   packet doesn’t match the filter it is allowed (not good). With AFM,
   if a packet does not match criteria, the packet is dropped.

-  AFM is a stateful packet inspection (SPI) firewall. This means that
   BIG-IP is aware of new packets coming to/from BIG-IP, existing
   packets, and rogue packets.

-  AFM adds more than 100 L2-4 denial of service attack vector
   detections and mitigations. This may be combined with ASM to provide
   L4-7 protection.

-  Application Delivery Firewall is the service defense in depth
   layering mentioned earlier. On top of a simple L4 network firewall,
   you may add access policy and controls from L4-7 with APM (Access
   Policy Manager), or add L7 deep packet inspection with ASM (web
   application firewall), You can add DNS DOS mitigation with LTM DNS
   Express and GTM + DNSSEC. These modules make up the entire
   Application Delivery Firewall (ADF) solution.

Creating AFM Network Firewall Rules
-----------------------------------

Default Actions
~~~~~~~~~~~~~~~

The BIG-IP\ :sup:`®` Network Firewall provides policy-based access
control to and from address and port pairs, inside and outside of your
network. Using a combination of contexts, the network firewall can apply
rules in many ways, including: at a global level, on a per-virtual
server level, and even for the management port or a self IP address.
Firewall rules can be combined in a firewall policy, which can contain
multiple context and address pairs, and is applied directly to a virtual
server.

By default, the Network Firewall is configured in **ADC mode**, a
default allow configuration, in which all traffic is allowed through the
firewall, and any traffic you want to block must be explicitly
specified.

The system is configured in this mode by default so all traffic on your
system continues to pass after you provision the Advanced Firewall
Manager. You should create appropriate firewall rules to allow necessary
traffic to pass before you switch the Advanced Firewall Manager to
Firewall mode. In **Firewall mode**, a default deny configuration, all
traffic is blocked through the firewall, and any traffic you want to
allow through the firewall must be explicitly specified.

The BIG-IP\ :sup:`®` Network Firewall provides policy-based access
control to and from address and port pairs, inside and outside of your
network. By default, the network firewall is configured in ADC mode,
which is a **default allow** configuration, in which all traffic is
allowed to virtual servers and self IPs on the system, and any traffic
you want to block must be explicitly specified. This applies only to the
Virtual Server & Self IP level on the system.

**Important: Even though the system is in a default allow configuration,
if a packet matches no rule in any context on the firewall, a Global
Drop rule drops the traffic.**

Rule Hierarchy
--------------

With the BIG-IP\ :sup:`®` Network Firewall, you use a context to
configure the level of specificity of a firewall rule or policy. For
example, you might make a global context rule to block ICMP ping
messages, and you might make a virtual server context rule to allow only
a specific network to access an application.

Context is processed in this order:

-  Global

-  Route domain

-  Virtual server / self IP

-  Management port\*

-  Global drop\*

The firewall processes policies and rules in order, progressing from the
global context, to the route domain context, and then to either the
virtual server or self IP context. Management port rules are processed
separately, and are not processed after previous rules. Rules can be
viewed in one list, and viewed and reorganized separately within each
context. You can enforce a firewall policy on any context except the
management port. You can also stage a firewall policy in any context
except management.

**Important: You cannot configure or change the Global Drop context. The
Global Drop context is the final context for traffic. Note that even
though it is a global context, it is not processed first, like the main
global context, but last. If a packet matches no rule in any previous
context, the Global Drop rule drops the traffic.**

|http://support.f5.com/kb/global/manual_images/MAN-0439-01/firewall_processing.png|

Create and View Log Entries
---------------------------

In this section, you will generate various types of traffic through the
firewall as you did previously, but now you will view the log entries
using the network firewall log. Open your web browser and once again try
to access http://10.30.0.50. Also, try to ping 10.30.0.50.

Open the **Security > Event Logs > Network > Firewall** page on
bigip2.dnstest.lab (192.168.1.150). The log file shows the ping requests
are being accepted and the web traffic is being dropped:

|image6|

Although we will not configure external logging in this lab, you should
be aware that the BIG-IP supports high speed external logging in various
formats including **SevOne**, **Splunk** and **ArcSight**.

Create a Rule List
~~~~~~~~~~~~~~~~~~

Rule lists are a way to group a set of individual rules together and
apply them to the active rule base as a group. A typical use of a rule
list would be for a set of applications that have common requirements
for access protocols and ports. As an example, most web applications
would require TCP port 80 for HTTP and TCP port 443 for SSL/TLS. You
could create a Rule list with these protocols, and apply them to each of
your virtual servers.

Let’s examine some of the default rule lists that are included with AFM.

Go to **Security >Network Firewall > Rule Lists**. They are:

-  \_sys_self_allow_all

-  \_sys_self_allow_defaults

-  \_sys_self_allow_management

|image7|

If you click on **\_sys_self_allow_management** you’ll see that it is
made up of two different rules that will allow management traffic (port
22/SSH and port 443 HTTPS). Instead of applying multiple rules over and
over across multiple servers, you can put them in a rule list and then
apply the rule list as an ACL.

|image8|

On bigip2.dnstest.lab (192.168.1.150) create a rule list to allow Web
traffic. A logical container must be created before the individual rules
can be added. You will create a list with two rules, to allow port 80
(HTTP) and reject traffic from a specific IP subnet. First you need to
create a container for the rules by going to:

**Security > Network Firewall > Rule Lists** and select **Create.**

For the **Name** enter **web_rule_list**, provide an optional
description and then click **Finished.**

|image9|

Edit the **web_rule_list** by selecting it in the Rule Lists table, then
click the **Add** button in the Rules section. Here you will add two
rules into the list; the first is a rule to allow HTTP.

|image10|

+-------------------------+------------------------------------------------+
| **Name**                | allow_http                                     |
+=========================+================================================+
| **Protocol**            | TCP                                            |
+-------------------------+------------------------------------------------+
| **Source**              | Leave at Default of **Any**                    |
+-------------------------+------------------------------------------------+
| **Destination Address** | **Specify...**\ 10.30.0.50, then click **Add** |
+-------------------------+------------------------------------------------+
| **Destination Port**    | **Specify…** Port **80**, then click **Add**   |
+-------------------------+------------------------------------------------+
| **Action**              | **Accept-Decisively**                          |
+-------------------------+------------------------------------------------+
| **Logging**             | Enabled                                        |
+-------------------------+------------------------------------------------+

|image11|\ r

Select **Repeat** when done.

Create another rule to reject all access from the 10.20.0.0/24 network.

+-------------------------+-------------------------------------+
| **Name**                | reject_10_20_0_0                    |
+=========================+=====================================+
| **Protocol**            | Any                                 |
+-------------------------+-------------------------------------+
| **Source**              | **Specify…**\ Address 10.20.0.0/24, |
|                         | then click **Add**                  |
+-------------------------+-------------------------------------+
| **Destination Address** | Any                                 |
+-------------------------+-------------------------------------+
| **Destination Port**    | Any                                 |
+-------------------------+-------------------------------------+
| **Action**              | Reject                              |
+-------------------------+-------------------------------------+
| **Logging**             | Enabled                             |
+-------------------------+-------------------------------------+

Select **Finished** when completed. When you exit, you’ll notice the
reject rule is after the **allow_http** rule. This means that HTTP
traffic from 10.20.0.0/24 will be accepted, while all other traffic from
this subnet will be rejected based on the ordering of the rules as seen
below:

|image12|

Create a Policy with a Rule List
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Policies are a way to group a set of individual rules together and apply
them to the active policy base as a group. A typical use of a policy
list would be for a set of rule lists that have common requirements for
access protocols and ports.

Create a policy list to allow the traffic you created in the rule list
in the previous section. A logical container must be created before the
individual rules can be added. First you need to create a container for
the policy by going to:

**Security > Network Firewall > Policies** and select **Create.**

You’ll notice that before Milton detached from Initech, he created a
global policy named **‘Global’** to allow basic connectivity to make
troubleshooting easier.

For the **Name** enter **rd_0_policy**, provide an optional description
and then click **Finished.
(Note: We commonly use “RD” in our rules to help reference the “Route
Domain”, default is 0)**

|image13|

Edit the **rd_0_policy** by selecting it in the Policy Lists table, then
click the **Add Rule List** button. Here you will add the rule list you
created in the previous section. For the **Name,** start typing
**web_rule_list**, you will notice the name will auto complete, select
the rule list **/Common/web_rule_list**, provide an optional description
and then click **Done Editing.**

|image14|

When finished your policy should look like the screen shot below.

|image15|

You will notice the changes are unsaved and need to be committed to the
system. This is a nice feature to have enabled to verify you want to
commit the changes you’ve just made without a change automatically being
implemented.

To commit the change, simply click **“Commit** Changes **to System”**
located at the top of the screen.

|image16|

Once committed you’ll notice the rule now becomes active and the
previous commit warning is removed.

|image17|

Add the Rule List to a Route Domain
-----------------------------------

In this section, you are going to attach the rule to a route domain
using the **Security** selection in the top bar within the **Route
Domain** GUI interface.

Go to **Network**, then click on **Route Domains**, then select the
hyperlink for route domain **0**.

Now click on the **Security** top bar selection, which is a new option
that was added in version 11.3.

In the Network Firewall section, set the Enforcement: to **“Enabled…”.**

Select the Policy you just created, “\ **rd_0_policy**\ ” and click
Update.

|image18|

Review the rules that are now applied to this route domain by navigating
to:

**Security > Network Firewall > Active Rules.**

From the **Context Filter** select **Route Domain 0**. You can expand
the web_rule_list by clicking the plus sign, your screen should look
similar to the below screen shot.

|image19|

Test the New Firewall Rules
---------------------------

Once again you will generate traffic through the BIG-IP AFM and then
view the AFM (firewall) logs.

-  Ping 10.30.0.50

-  Open a new Web browser and access http://10.30.0.50

-  Open a new Web browser and access http://10.30.0.50:8081

-  SSH to 10.30.0.50 using Web Server shortcut (PUTTY) on desktop.

In the Configuration Utility, open the **Security > Event Logs > Network
> Firewall** page.

Access for port 80 was granted to a host using the web_rule_list:
**allow_http** rule.

|image20|

Requests for port 8081, and 22 were all rejected due to the
reject_10_20_0_0 rule.

|image21|

You may verify this, by going to **Security > Network Firewall > Active
Rules**, then selecting the context for route domain 0. Note the
**Count** field next to each rule as seen below. Also note how each rule
will also provide a **Latest Matched** field so you will know the last
time each rule was matched:

|image22|

| Congratulations! Day one and you’ve already saved the day. Hang on,
  something isn’t right, the images Mr. Lumbergh talked about are not
  populating, they look like broken links.
| |image23|

Let’s refresh the web page once more and see what the logs show….

|image24|

If we follow the flow we can see the traffic to 10.30.0.50 is permitted
on port 80 however; there appears to be a second connection attempting
to open to another server, 10.40.0.50, also on port 80 (glad we put in
that reject rule and are logging all the traffic flows). Let’s look at
how this web page is written. To view the page source details, simply
**right** click anywhere on the **10.30.0.50** web page and select “view
page source”

re\ |image25|

Very interesting, it appears there are two images and they are links to
another server which appear to be a server on the application network,
which is also a link off of the firewall. You can verify this by looking
at the network settings on the BIG-IP found under: **Network > VLANs
and/or Network > Self IPs.** To resolve, let’s create another rule list
for this network as well to keep the rule lists separated for security
reasons.

Creating an Additional Rule List for Additional Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Rules and Rule Lists can also be created and attached to a context from
the Active Rules section of the GUI. Go to the

**Security > Network Firewall > Rule Lists**

Create a **Rule List** called **application_rule_list** then click
**Finished**.

Enter the rule list by clicking on its hyperlink, then in the **Rules**
section click **Add**, and add the following information, then click
**Finished**.

+-------------------------+-----------------------------------------------+
| **Name**                | allow_http                                    |
+=========================+===============================================+
| **Protocol**            | TCP                                           |
+-------------------------+-----------------------------------------------+
| **Source**              | Leave at Default of **Any**                   |
+-------------------------+-----------------------------------------------+
| **Destination Address** | **Specify…**\ 10.40.0.50, then click **Add**  |
+-------------------------+-----------------------------------------------+
| **Destination Port**    | **Specify…**\ Port **80**, then click **Add** |
+-------------------------+-----------------------------------------------+
| **Action**              | **Accept-Decisively**                         |
+-------------------------+-----------------------------------------------+
| **Logging**             | Enabled                                       |
+-------------------------+-----------------------------------------------+

|image26|

Add Another Rule List to the Policy
-----------------------------------

Use the **Policies** page to add the new firewall rule list to the
**rd_0_policy**.

Open the **Security > Network Firewall > Policies** page.

Click on the policy name to modify the policy.

The only current active rule list is for the **web_policy**. Click on
the arrow next to **Add Rule List, then select, Add the rule list AT
END)** to add the new rule list you just created. For **Name** begin
typing ‘application_rule_list’, select /Common/application_rule_list,
then click **Done Editing**.

Remember to Commit the changes to system before proceeding.

Once completed, you should see a policy similar to the one below:

|image27|

Test Access to the Server
-------------------------

-  Open a new Web browser and access http://10.30.0.50

Good to, wait, not go. What happened? I added a rule, why didn’t this
work? Let’s look at the logs again (**Security > Event Logs > Network >
Firewall).** They basically look the same as before, lets look at the
ordering of the rule we just created (**Security > Network Firewall >
Active Rules change contex to route domain 0).** Take note the newly
created rule has a counter value of 0, if we look at the order we can
see the reject rule, which we added in the web_rule_list has incremented
and appears to be matching the traffic before it reaches our new rule.
**(Be sure to expand the Rule List to see the counts)** Let’s modify the
rule order slightly to accomplish what we’re looking for. From within
the Active Rules section simply drag the application_rule_list **ABOVE**
the web_rule_list. Don’t forget to commit the changes.

The new ordering should look something like the screen shot below:

|image28|

.. _test-access-to-the-server-1:

Test Access to the Server
-------------------------

-  Open a new Web browser and access http://10.30.0.50

**Success!!**

Before we continue let’s clean up the rules just a little for best
practices. The clean-up/catch-all/drop/etc rule is typically applied to
the end of your policy, not necessarily within the rule-list. While its
perfectly acceptable to have drop statements within individual rules to
prevent certain traffic, the broader drop statement should be applied at
the end of the policy (remember how AFM processes contexts from the
beginning of this lab – see pages 6+7).

Use the **Rule Lists** page to modify the firewall rule
**‘web_rule_list’**. Open the **Security > Network Firewall > Rule
Lists** page. Click on the rule list **‘web_rule_list’** to modify the
rule list. Check the box next to the reject_10_20_0_0 rule and click
‘\ **Remove’.** The updated rule should look something like the below
screen shot:

|image29|

Next, you’ll want to add the reject rule to the policy. In the
Configuration Utility, open the **Security > Network Firewall >
Policies** page. Click on the **rd_0_policy**. Select ‘Add Rule’ drop
down and select at the end. You’ll notice all the same options are
available within a policy as they are within a rule-list. Create an
entry with the following information then click Done Editing and commit
the change.

+-------------------------+------------------------------------------+
| **Name**                | reject_10_20_0_0                         |
+=========================+==========================================+
| **Protocol**            | Any                                      |
+-------------------------+------------------------------------------+
| **Source**              | Address 10.20.0.0/24, then click **Add** |
+-------------------------+------------------------------------------+
| **Destination Address** | Any                                      |
+-------------------------+------------------------------------------+
| **Destination Port**    | Any                                      |
+-------------------------+------------------------------------------+
| **Action**              | Reject                                   |
+-------------------------+------------------------------------------+
| **Logging**             | Enabled                                  |
+-------------------------+------------------------------------------+

The new Policy should look something like the screen shot below:

|image30|

.. _test-the-new-firewall-rules-1:

Test the New Firewall Rules
---------------------------

Once again you will generate traffic through the BIG-IP AFM and then
view the AFM (firewall) logs.

-  Ping 10.30.0.50

-  Open a new Web browser and access http://10.30.0.50

-  Open a new Web browser and access http://10.30.0.50:8081

-  SSH to 10.30.0.50 using Web Server shortcut on desktop

In the Configuration Utility, open the **Security > Event Logs > Network
> Firewall** page.

Access for port 80 on 10.30.0.50 was granted using the web_rule_list:
**allow_http** rule.

|image31|

Access for port 80 on 10.40.0.50 was granted using the
application_rule_list: **allow_http** rule.\ |image32|

Ping to 10.30.0.50 was granted using the global rule.

|image33|

All other traffic was rejected by the rd_0_policy reject_10_20_0_0
reject rule

|image34|

View Firewall Reports
---------------------

View several of the built-in network firewall reports and graphs on the
BIG-IP system. Open the **Security >Reporting > Network > Enforced
Rules** page. The default report shows all the rule contexts that were
matched in the past hour.

|image35|

The default view gives reports per Context, in the drop-down menu select
**Rules (Enforced)**.

|image36|

From the **View By** list, select **Destination Ports (Enforced)**.

|image37|

This redraws the graph to report more detail for all the destination
ports that matched an ACL.

|image38|

From the **View By** list, select **Source IP Addresses (Enforced)**.
This shows how source IP addresses matched an ACL clause:

|image39|

**AFM Reference Material
**

-  **Network World Review of AFM: F5 data center firewall aces
   performance test
   **\ http://www.networkworld.com/reviews/2013/072213-firewall-test-271877.html

-  **AFM Product Details
   on**\ *\ *\ `www.f5.com <http://www.f5.com>`__\ *\ *\ **
   **\ http://www.f5.com/products/big-ip/big-ip-advanced-firewall-manager/overview

-  **AFM Operations Guide**

   https://support.f5.com/content/kb/en-us/products/big-ip-afm/manuals/product/f5-afm-operations-guide/_jcr_content/pdfAttach/download/file.res/f5-afm-operations-guide.pdf

|image40|

**F5 BIG-IP Training**

**Advanced Firewall Manager**

**AFM Packet Tester, Flow Inspector, Stale Rule Lab - 02**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

.. _lab-overview-1:

Lab Overview
============

New in the v13 release of the BIG-IP Advanced Firewall Manager is the capability to insert a packet trace into the internal flow so you can analyze what component within the system is allowing or blocking packets based on your configuration of features and rule sets.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

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

**F5 BIG-IP Training**

**Advanced Firewall Manager**

**AFM DDoS Lab - 03**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

.. _lab-overview-2:

Lab Overview
============

During this lab, you will configure the BIG-IP system to detect and
report on various network level Denial of Service events. You will then
run simulated attacks against the BIG-IP and verify the mitigation,
reporting and logging of these attacks.

.. _base-big-ip-configuration-1:

Base BIG-IP Configuration
-------------------------

In this lab, the VE has been configured with the basic system settings
and the VLAN/self-IP configurations required for the BIG-IP to
communicate and pass traffic on the network. We will now need to
configure the BIG-IP to listen for traffic and pass it to the back-end
server.

Lab 1 – Advanced Firewall Manager (AFM) DNS DoS
===============================================

It is day two of your career at Initech, and you are under attack!! You
walk into the office on day two only to learn your DNS servers are being
attacked by Joanna who took out her flair frustrations on your DNS
servers. Before you can protect the servers however, you must first tune
and configure them appropriately. (The most challenging part of DoS
based protection is tuning correctly).

1.  Launch the Chrome shortcut titled “BIG-IP UI” on the desktop of your
    lab jump server. For this lab you will be working on
    bigip1.dnstest.lab (http://192.168.1.100). The credentials for the
    BIG-IP are conveniently displayed in the login banner. Just in case:
    **admin / 401elliottW!**

2.  Navigate to **Local Traffic** > **Nodes** and create a new node with
    the following settings, leaving unspecified fields at their default
    value:

    a. Name: lab-server-10.10.0.50

    b. | Address: 10.10.0.50
       | |image51|

3.  Click **Finished** to add the new node.

4.  Navigate to **Local Traffic** > **Pools** and create a new pool with
    the following settings, leaving unspecified attributes at their
    default value:

    c. Name: lab-server-pool

    d. Health Monitors: gateway_icmp

    e. New Members: Node List

       i.   Address: lab-server-10.10.0.50

       ii.  Service Port: \* (All Services)

       iii. | Click **Add** to add the new member to the member list.
            | |image52|

5.  Click **Finished** to create the new pool.

6.  Because the attack server will be sending a huge amount of traffic,
    we’ll need a large SNAT pool. Navigate to **Local Traffic** >
    **Address Translation** > **SNAT Pool List** and create a new SNAT
    pool with the following attributes:

    f. Name: inside_snat_pool

    g. | Member List (click Add after each IP):
       | 10.10.0.125, 10.10.0.126, 10.10.0.127, 10.10.0.128,
         10.10.0.129, 10.10.0.130

    h. | Click Finished
       | |image53|

7.  Navigate to **Local Traffic** > **Virtual Servers** and create a new
    virtual server with the following settings, leaving unspecified
    fields at their default value:

    i. Name: udp_dns_VS

    j. Destination Address/Mask: 10.20.0.10

    k. Service Port: 53 (other)

    l. Protocol: UDP

    m. Source Address Translation: SNAT

    n. SNAT Pool: inside_snat_pool

    o. Default Pool: lab-server-pool

8.  Click Finished

    |image54|

9.  We’ll now test the new DNS virtual server. SSH into the attack host
    by clicking the “Attack Host (Ubuntu)” icon on the jump host
    desktop.

10. Issue the dig @10.20.0.10
    `www.example.com <http://www.example.com>`__ +short command on the
    BASH CLI of the attack host. You should see output similar to:
    |image55|\ This verifies that DNS traffic is passing through the
    BIG-IP.

11. Return to the BIG-IP and navigate to **Local Traffic** > **Virtual
    Servers** and create a new virtual server with the following
    settings, leaving unspecified fields at their default value:

    p. Name: other_protocols_VS

    q. Destination Address/Mask: 10.20.0.10

    r. Service Port: \* (All Ports)

    s. Protocol: \* All Protocols

    t. Any IP Profile: ipother

    u. Source Address Translation: SNAT

    v. SNAT Pool: inside_snat_pool

    w. Default Pool: lab-server-pool

12. | Click Finished
    | |image56|

13. Return to the Attack Host SSH session and attempt to SSH to the
    server using SSH 10.20.0.10. Simply verify that you are prompted for
    credentials and press CTRL+C to cancel the session. This verifies
    that non-DNS traffic is now flowing through the BIG-IP.

Detecting and Preventing DNS DoS Attacks on a Virtual Server
------------------------------------------------------------

Establishing a DNS server baseline
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before we can prevent Joanna from attacking our DNS server, again, we
should establish a baseline for how many QPS our DNS server can handle.
For this lab, let’s find the magic number of QPS that causes 50% CPU
utilization on the BIND process.

1. Connect to the Victim Server SSH session by double-clicking the
   **Victim Server (Ubuntu)** shortcut on the jump host desktop.

2. From the BASH prompt, enter **top** and press **Enter** to start the
   top utility.

3. You will see a list of running processes sorted by CPU utilization,
   like the output below:

   |image57|

4. Connect to the Attack Host SSH session by double-clicking the
   **Attack Host (Ubuntu)** shortcut on the jump host desktop.

5. | Start by sending 500 DNS QPS for 30 seconds to the host using the
     following syntax:
   | dnsperf -s 10.20.0.10 -d queryfile-example-current -c 20 -T 20 -l
     30 -q 10000 -Q 500

6. Observe CPU utilization over the 30 second window for the **named**
   process. If the CPU utilization is below 45%, increase the QPS by
   increasing the -Q value. If the CPU utilization is above 55%,
   decrease the QPS. This

7. Record the QPS required to achieve a sustained CPU utilization of
   approximately 50%. Consider this the QPS that the server can safely
   sustain for demonstration purposes.

8. | Now, attack the DNS server with 10,000 QPS using the following
     syntax:
   | dnsperf -s 10.20.0.10 -d queryfile-example-current -c 20 -T 20 -l
     30 -q 10000 -Q 10000

9. You’ll notice that the CPU utilization on the victim server
   skyrockets, as well as DNS query timeout errors appearing on the
   attack server’s SSH session. This shows your DNS server is
   overwhelmed.

Configuring a DoS Logging Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We’ll create a DoS logging profile so that we can see event logs in the
BIG-IP UI during attack mitigation.

1. On the BIG-IP web UI, navigate to **Security** > **Event Logs** >
   **Logging Profiles** and create a new profile with the following
   values, leaving unspecified attributes at their default value:

   a. Profile Name: dns-dos-profile-logging

   b. DoS Protection: Enabled

   c. | DNS DoS Protection Publisher: local-db-publisher and click
        finish
      | |image58|

.. _section-3:

.. _section-4:

Configuring a DoS Profile
~~~~~~~~~~~~~~~~~~~~~~~~~

We will now create a DoS profile with manually configured thresholds to
limit the attack’s effect on our server.

1. Navigate to **Security** > **DoS Protection** > **DoS Profiles**

2. Create a new DoS profile with the name **dns-dos-profile**.

3. | Click **Finished**.
   | |image59|

4. The UI will return to the DoS Profiles list. Click the
   **dns-dos-profile** name.

5. Click the **Protocol Security** tab and select **DNS Security** from
   the drop-down.

6. Click the **DNS A Query** vector from the Attack Type list.

7. Modify the **DNS A Query** vector configuration to match the
   following values, leaving unspecified attributes with their default
   value:

   a. State: Mitigate

   b. Threshold Mode: Fully Manual

   c. Detection Threshold EPS: (Set this at 80% of your safe QPS value)

   d. | Mitigation Threshold EPS: (Set this to your safe QPS value)
      | |image60|

8. Make sure that you click **Update** to save your changes.

Attaching a DoS Profile
~~~~~~~~~~~~~~~~~~~~~~~

We will attach the DoS profile to the virtual server that we configured
to manage DNS traffic.

1. Navigate to **Local Traffic** > **Virtual Servers** > **Virtual
   Server List**.

2. Click on the **udp_dns_VS** name.

3. Click on the **Security** tab and select **Policies**.

4. In the **DoS Protection Profile** field, select **Enabled** and
   choose the **dns-dos-profile**.

5. In the **Log Profile**, select **Enabled** and move the
   **dns-dos-profile-logging** profile from **Available** to
   **Selected**.

6. Click **Update**.

Simulate a DNS DDoS Attack
~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Open the SSH session to the victim server and ensure the top utility
   is running.

2. | Once again, attack your DNS server from the attack host using the
     following syntax:
   | dnsperf -s 10.20.0.10 -d queryfile-example-current -c 20 -T 20 -l
     30 -q 10000 -Q 10000

3. On the server SSH session running the top utility, notice the CPU
   utilization on your server remains in a range that ensures the DNS
   server is not overwhelmed.

4. After the attack, navigate to **Security** > **Event Logs** > **DoS**
   > **DNS Protocol**. Observe the logs to see the mitigation actions
   taken by the BIG-IP. Be sure to scroll right…

   |image61|

DNS DDoS Mitigations for Continued Service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At this point, you have successfully configured the BIG-IP to limit the
amount of resource utilization on the BIG-IP, thus further frustrating
Joanna on her flair rage. Unfortunately, even valid DNS requests can be
caught in the mitigation we’ve configured. There are further steps that
can be taken to mitigate Joanna’s attack that will allow non-malicious
DNS queries.

Bad Actor Detection
^^^^^^^^^^^^^^^^^^^

Bad actor detection and blacklisting allows us to completely block
communications from malicious hosts at the BIG-IP, completely preventing
those hosts from reaching the back-end servers. To demonstrate:

1.  Navigate to **Security** > **DoS Protection** > **DoS Profiles**.

2.  Click on the **dns-dos-profile** profile name.

3.  Click on the **Protocol Security** tab then select **DNS Security**.

4.  Click on the **DNS A Query** attack type name.

5.  Modify the vector as follows:

    a. Bad Actor Detection: Checked

    b. Per Source IP Detection Threshold EPS: 80

    c. Per Source IP Mitigation Threshold EPS: 100

    d. Add Source Address to Category: Checked

    e. Category Name: denial_of_service

    f. Sustained Attack Detection Time: 15 seconds

    g. | Category Duration Time: 60 seconds
       | |image62|

6.  Make sure you click **Update** to save your changes.

7.  Navigate to **Security** > **Network Firewall** > **IP
    Intelligence** > **Policies** and create a new IP Intelligence
    policy with the following values, leaving unspecified attributes at
    their default values:

    h. Name: dns-bad-actor-blocking

    i. Default Log Actions section:

       i. Log Blacklist Category Matches: Yes

    j. Blacklist Matching Policy

       ii. Create a new blacklist matching policy:

           1. Blacklist Category: denial_of_service

           2. |image63|\ Click **Add** to add the policy then click
              finished

8.  Navigate to **Local Traffic** > **Virtual Servers** > **Virtual
    Server List**.

9.  Click on the **udp_dns_VS** virtual server name.

10. Click on the **Security** tab and select **Policies**.

11. | Enable **IP Intelligence** and choose the
      **dns-bad-actor-blocking** policy.
    | |image64|

12. Make sure you click **Update** to save your changes.

13. Navigate to **Security** > **Event Logs** > **Logging Profiles**.

14. Click the **global-network** logging profile name.

15. | Under the **Network Firewall** tab (next to Protocol Security),
      set the IP Intelligence Publisher to **local-db-publisher** and
      check **Log Shun Events**.
    | |image65|

16. Click **Update** to save your changes.

17. Click the **dns-dos-profile-logging** logging profile name.

18. | Check **Enabled** next to **Network Firewall**.
    | |image66|

19. | Under the **Network Firewall** tab, change the **IP Intelligence
      Publisher** to **local-db-publisher** and click **Update**.
    | |image67|

20. Bring into view the Victim Server SSH session running the top
    utility to monitor CPU utilization.

21. | On the Attack Server host, launch the DNS attack once again using
      the following syntax:
    | dnsperf -s 10.20.0.10 -d queryfile-example-current -c 20 -T 20 -l
      30 -q 10000 -Q 10000

22. | You’ll notice CPU utilization on the BIG-IP begin to climb, but
      slowly drop. The attack host will show that queries are timing out
      as shown below. This is due to the BIG-IP blacklisting the bad
      actor.
    | |image68|

23. Navigate to **Security** > **Event Logs** > **Network** > **IP
    Intelligence**. Observe the bad actor blocking mitigation logs.

24. | Navigate to **Security** > **Event Logs** > **Network** >
      **Shun**. This screen shows the bad actor being added to (and
      later deleted from) the shun category.
    | |image69|

25. While the attack is running, navigate to **Security** > **DoS
    Protection**> **DoS Overview (you may need to refresh or set the
    auto refresh to 10 seconds).** You will notice from here you can see
    all the details of the active attacks. You can also modify an attack
    vector right from this screen by clicking on the attack vector and
    modifying the fly out.

|image70|

26. | Navigate to **Security** > **Reporting** > **Protocol** > **DNS**.
      Change the **View By** drop-down to view various statistics around
      the DNS traffic and attacks.
    | |image71|

27. Navigate to **Security** > **Reporting** > **Network** > **IP
    Intelligence**. The default view may be blank. Change the **View
    By** drop-down to view various statistics around the IP Intelligence
    handling of the attack traffic.

28. | Navigate to **Security** > **Reporting** > **DoS** > **Dashboard**
      to view an overview of the DoS attacks and timeline. You can
      select filters in the filter pane to highlight specific attacks.
    | |image72|

29. Finally, navigate to **Security** > **Reporting** > **DoS** >
    **Analysis**. View detailed statistics around each attack.

|image73|

Remote Triggered Black Holing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The BIG-IP supports the advertisement of bad actor(s) to upstream
devices via BGP to block malicious traffic closer to the source. This is
accomplished by publishing a blacklist to an external resource. This is
not demonstrated in this lab.

Silverline Mitigation
^^^^^^^^^^^^^^^^^^^^^

F5’s Silverline service offers “always on” and “on demand” DDoS
scrubbing that could assist in this scenario as well. This is not
demonstrated in this lab.

Filtering specific DNS operations
---------------------------------

The BIG-IP offers the ability to filter DNS query types and header
opcodes to act as a DNS firewall. To demonstrate, we will block MX
queries from our DNS server.

1.  Open the SSH session to the Attack Host.

2.  | Perform an MX record lookup by issuing the following command:
    | dig @10.20.0.10 MX example.com

3.  The server doesn’t have a record for this domain. This server
    doesn’t have MX records, so those requests should be filtered

4.  Navigate to **Security** > **Protocol Security** > **Security
    Profiles** > **DNS** and create a new DNS security profile with the
    following values, leaving unspecified attributes at their default
    value:

    a. Name: dns-block-mx-query

    b. | Query Type Filter: move mx from Available to Active and click
         finished
       | |image74|

5.  Navigate to **Local Traffic** > **Profiles** > **Services** >
    **DNS**. **NOTE:** if you are mousing over the services, DNS may not
    show up on the list. Select **Services** and then use the pulldown
    menu on services to select **DNS**.

6.  Create a new DNS services profile with the following values, leaving
    unspecified values at their default values:

    c. Name: dns-block-mx

    d. DNS Traffic

       i.  DNS Security: Enabled

       ii. | DNS Security Profile Name: dns-block-mx-query. Click
             finished
           | |image75|

7.  Navigate to **Local Traffic** > **Virtual Servers** > **Virtual
    Server List**.

8.  Click on the **udp_dns_VS** virtual server name.

9.  In the **Configuration** section, change the view to **Advanced**.

10. | Set the **DNS Profile** to **dns-block-mx**.
    | |image76|

11. Click **Update** to save your settings.

12. Navigate to **Security** > **Event Logs** > **Logging Profiles**.

13. Click on the **dns-dos-profile-logging** logging profile name.

14. Check **Enabled** next to **Protocol Security**.

15. | In the **Protocol Security** tab, set the **DNS Security
      Publisher** to **local-db-publisher** and check all five of the
      request log types.
    | |image77|

16. Make sure that you click **Update** to save your settings.

17. | Return to the Attack Server SSH session and re-issue the MX query
      command:
    | dig @10.20.0.10 MX example.com

18. The query hangs as the BIG-IP is blocking the MX lookup.

19. | Navigate to **Security** > **Event Logs** > **Protocol** >
      **DNS**. Observe the MX query drops.
    | |image78|

This concludes the DNS portion of the lab. On the Victim Server, stop
the top utility by pressing **CTRL + C**. No mail for you Joanna!!

Lab 2 – Advanced Firewall Manager (AFM) Detecting and Preventing System DoS and DDoS Attacks
--------------------------------------------------------------------------------------------

Configure Logging
~~~~~~~~~~~~~~~~~

Configuring a logging destination will allow you to verify the BIG-IPs
detection and mitigation of attacks, in addition to the built-in
reporting.

1. In the BIG-IP web UI, navigate to **Security** > **DoS Protection** >
   **Device Configuration** > **Properties**.

2. Under **Log Pubisher**, select **local-db-publisher**.

3. | Click the **Commit Changes to System** button.
   | |image79|

.. _section-5:

Simulating a Christmas Tree Packet Attack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Joanna was feeling festive this morning. In this example, we’ll set the
BIG-IP to detect and mitigate Joanna’s attack where all flags on a TCP
packet are set. This is commonly referred to as a Christmas Tree Packet
and is intended to increase processing on in-path network devices and
end hosts to the target.

We’ll use the hping utility to send 25,000 packets to our server, with
random source IPs to simulate a DDoS attack where multiple hosts are
attacking our server. We’ll set the SYN, ACK, FIN, RST, URG, PUSH, Xmas
and Ymas TCP flags.

1.  In the BIG-IP web UI, navigate to **Security** > **DoS Protection**
    > **Device Configuration** > **Network Security**.

2.  Expand the **Bad-Header-TCP** category in the vectors list.

3.  Click on the **Bad TCP Flags (All Flags Set)** vector name.

4.  Configure the vector with the following parameters:

    a. State: Mitigate

    b. Threshold Mode: Fully Manual

    c. Detection Threshold EPS: Specify 50

    d. Detection Threshold Percent: Specify 200

    e. | Mitigation Threshold EPS: Specify 100
       | |image80|

5.  Click **Update** to save your changes.

6.  Open the BIG-IP SSH session and scroll the ltm log in real time with
    the following command: tail -f /var/log/ltm

7.  | On the attack host, launch the attack by issuing the following
      command on the BASH prompt:
    | sudo hping3 10.20.0.10 --flood --rand-source --destport 80 -c
      25000 --syn --ack --fin --rst --push --urg --xmas --ymas

8.  | You’ll see the BIG-IP ltm log show that the attack has been
      detected:
    | |image81|

9.  | After approximately 60 seconds, press **CTRL+C** to stop the
      attack.
    | |image82|

10. Navigate to **Security** > **DoS Protection**> **DoS Overview (you
    may need to refresh or set the auto refresh to 10 seconds).** You’ll
    notice from here you can see all the details of the active attacks.
    You can also modify an attack vector right from this screen by
    clicking on the attack vector and modifying the details in the fly
    out panel.

    |image83|

11. | Return to the BIG-IP web UI. Navigate to **Security** > **Event
      Logs** > **DoS** > **Network** > **Events**. Observe the log
      entries showing the details surrounding the attack detection and
      mitigation.
    | |image84|

12. Navigate to **Security** > **Reporting** > **DoS** > **Analysis**.
    Single-click on the attack ID in the filter list to the right of the
    charts and observe the various statistics around the attack.

Simulating a TCP SYN DDoS Attack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the last example, Joanna crafted a packet that is easily identified
as malicious, as its invalid. We’ll now simulate an attack with traffic
that could be normal, acceptable traffic. The TCP SYN flood attack will
attempt to DDoS a host by sending valid TCP traffic to a host from
multiple source hosts.

1.  In the BIG-IP web UI, go to **Security** > **DoS Protection** >
    **Device** **Configuration** > **Network Security**.

2.  Expand the **Flood** category in the vectors list.

3.  Click on **TCP Syn Flood** vector name.

4.  Configure the vector with the following parameters:

    a. State: Mitigate

    b. Threshold Mode: Fully Manual

    c. Detection Threshold EPS: 200

    d. Detection Threshold Percent: 500

    e. | Mitigation Threshold EPS: 400
       | |image85|

5.  Click **Update** to save your changes.

6.  Open the BIG-IP SSH session and scroll the ltm log in real time with
    the following command: tail -f /var/log/ltm

7.  | On the attack host, launch the attack by issuing the following
      command on the BASH prompt:
    | sudo hping3 10.20.0.10 --flood --rand-source --destport 80 --syn
      -d 120 -w 64

8.  After about 60 seconds, stop the flood attack by pressing **CTRL +
    C**.

9.  Return to the BIG-IP web UI and navigate to **Security** > **Event
    Logs** > **DoS** > **Network** > **Events**. Observe the log entries
    showing the details surrounding the attack detection and mitigation.

10. Navigate to **Security** > **Reporting** > **DoS** > **Dashboard**
    to view an overview of the DoS attacks and timeline. You can select
    filters in the filter pane to highlight the specific attack.

11. Finally, navigate to **Security** > **Reporting** > **DoS** >
    **Analysis**. View detailed statistics around the attack.

Preventing Global DoS Sweep and Flood Attacks
---------------------------------------------

In the last section, the focus was on attacks originating from various
hosts. In this section, we will focus on mitigating flood and sweep
attacks from a single host.

Single Endpoint Sweep
^^^^^^^^^^^^^^^^^^^^^

The single endpoint sweep is an attempt for an attacker to send traffic
across a range of ports on the target server, typically to scan for open
ports.

1.  In the BIG-IP web UI, navigate to **Security** > **DoS Protection**
    > **Device Configuration** > **Network Security**.

2.  Expand the **Single-Endpoint** category in the vectors list.

3.  Click on **Single Endpoint Sweep** vector name.

4.  Configure the vector with the following parameters:

    a. State: Mitigate

    b. Threshold Mode: Fully Manual

    c. Detection Threshold EPS: 150

    d. Mitigation Threshold EPS: 200

    e. Add Source Address to Category: Checked

    f. Category Name: denial_of_service

    g. Sustained Attack Detection Time: 10 seconds

    h. Category Duration Time: 60 seconds

    i. | Packet Type: Move All IPv4 to Selected
       | |image86|

5.  Click **Update** to save your changes.

6.  Navigate to **Security** > **Network Firewall** > **IP
    Intelligence** > **Policies**.

7.  | In the **Global Policy** section, change the **IP Intelligence
      Policy** to **ip-intelligence**.
    | |image87|

8.  Click **Update**.

9.  Click on the **ip-intelligence** policy in the policy list below.

10. Create a new Blacklist Matching Policy in the IP Intelligence Policy
    Properties section with the following attributes, leaving
    unspecified attributes with their default values:

    j. Blacklist Category: denial-of-service

    k. Action: drop

    l. Log Blacklist Category Matches: Yes

11. | Click **Add** to add the new Blacklist Matching Policy.
    | |image88|

12. Click **Update** to save changes to the ip-intelligence policy.

13. Open the BIG-IP SSH session and scroll the ltm log in real time with
    the following command: tail -f /var/log/ltm

14. On the victim server, start a packet capture with an SSH filter by
    issuing sudo tcpdump -nn not port 22

15. | On the attack host, launch the attack by issuing the following
      command on the BASH prompt:
    | sudo hping3 10.20.0.10 --flood --scan 1-65535 -d 128 -w 64 --syn

16. You will see the scan find a few open ports on the server, and the
    server will show the inbound sweep traffic. However, you will notice
    that the traffic to the server stops after a short time (10 seconds,
    the configured sustained attack detection time.) Leave the test
    running.

17. After approximately 60 seconds, sweep traffic will return to the
    host. This is because the IP Intelligence categorization of the
    attack host has expired. After 10 seconds of traffic, the bad actor
    is again blacklisted for another 60 seconds.

18. Stop the sweep attack on the attack host by pressing **CTRL + C**.

19. Return to the BIG-IP web UI and navigate to **Security** > **Event
    Logs** > **DoS** > **Network** > **Events**. Observe the log entries
    showing the details surrounding the attack detection and mitigation.

20. Navigate to **Security** > **Event Logs** > **Network** > **IP
    Intelligence**. Observe the log entries showing the mitigation of
    the sweep attack via the ip-intelligence policy.

21. Navigate to **Security** > **Event Logs** > **Network** > **Shun**.
    Observe the log entries showing the blacklist adds and deletes.

22. Navigate to **Security** > **Reporting** > **Network** > **IP
    Intelligence**. Observe the statistics showing the sweep attack and
    mitigation. Change the **View By** drop-down to view the varying
    statistics.

23. Navigate to **Security** > **Reporting** > **DoS** > **Dashboard**
    to view an overview of the DoS attacks and timeline. You can select
    filters in the filter pane to highlight the specific attack.

24. Finally, navigate to **Security** > **Reporting** > **DoS** >
    **Analysis**. View detailed statistics around the attack.

Single Endpoint Flood
^^^^^^^^^^^^^^^^^^^^^

The single endpoint flood attack is an attempt for an attacker to send a
flood of traffic to a host in hopes of overwhelming a service to a point
of failure. In this example, we’ll flood the target server with ICMP
packets.

1.  In the BIG-IP web UI, navigate to **Security** > **DoS Protection**
    > **Device Configuration** > **Network Security**.

2.  Expand the **Single-Endpoint** category in the vectors list.

3.  Click on **Single Endpoint Flood** vector name.

4.  Configure the vector with the following parameters:

    a. State: Mitigate

    b. Threshold Mode: Fully Manual

    c. Detection Threshold EPS: 150

    d. Mitigation Threshold EPS: 200

    e. Add Destination Address to Category: Checked

    f. Category Name: denial_of_service

    g. Sustained Attack Detection Time: 10 seconds

    h. Category Duration Time: 60 seconds

    i. | Packet Type: Move Any ICMP (IPv4) to Selected
       | |image89|

5.  Click **Update** to save your changes.

6.  Open the BIG-IP SSH session and scroll the ltm log in real time with
    the following command: tail -f /var/log/ltm

7.  We’ll run a packet capture on the victim server to gauge the
    incoming traffic. On the victim server, issue the following command:
    sudo tcpdump -nn not port 22

8.  | On the attack host, launch the attack by issuing the following
      command on the BASH prompt:
    | sudo hping3 10.20.0.10 --faster -c 25000 --icmp

9.  The attack host will begin flooding the victim server with ICMP
    packets. However, you will notice that the traffic to the server
    stops after a short time (10 seconds, the configured sustained
    attack detection time.)

10. After approximately 60 seconds, run the attack again. ICMP traffic
    will return to the host. This is because the IP Intelligence
    categorization of the attack host has expired.

11. Return to the BIG-IP web UI.

12. Navigate to **Security** > **Event Logs** > **DoS** > **Network** >
    **Events**. Observe the log entries showing the details surrounding
    the attack detection and mitigation.

13. Navigate to **Security** > **Event Logs** > **Network** > **IP
    Intelligence**. Observe the log entries showing the mitigation of
    the sweep attack via the ip-intelligence policy.

14. Navigate to **Security** > **Reporting** > **Network** > **IP
    Intelligence**. Observe the statistics showing the sweep attack and
    mitigation.

15. Navigate to **Security** > **Reporting** > **DoS** > **Dashboard**
    to view an overview of the DoS attacks and timeline. You can select
    filters in the filter pane to highlight the specific attack.

16. Finally, navigate to **Security** > **Reporting** > **DoS** >
    **Analysis**. View detailed statistics around the attack.

This concludes the DoS/DDoS portion of the lab. You have successfully
defeated Joanna, she has decided a career at Chotchkie’s is more
prosperous than nefarious internet activities, even with the new flair
requirements. Well done!

|image90|

**F5 BIG-IQ Training**

**Device Management Workflows**

**Device Management Lab - 04**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

|http://cdn.ws.citrix.com/wp-content/uploads/2011/08/ICSA_Cert_WebFirewall_2C_200DPI_720x374.jpg|

.. _section-6:

.. _lab-overview-3:

Lab Overview
============

Day 3, you get a little curious and wonder why both BIG-IP’s you’ve been
working on say they’re managed by BIG-IQ (look near the red f5 ball on
the top left of both BIG-IP’s). Unbelievable, all this time you’ve been
configuring both devices independently when you could have been
configuring them on a central management device.

Central Management Version - 6.0 was a major evolution of the BIG-IQ
product line designed to become the primary source of centralized
management for all physical and virtual F5 BIG-IP devices. BIG-IQ
extends its offerings for security users, improving the user experience,
and adding robustness and scale throughout the platform. 

Base BIG-IQ Configuration
-------------------------

In this lab, the VE has been configured with the basic system settings
and the VLAN/self-IP configurations required for the BIG-IQ to
communicate and pass traffic on the network. Additionally, the Data
Collection Device has already been added to BIG-IQ and the BIG-IP’s have
been imported and have been gathering health statistics. They have not
however had their configurations imported.

New features
------------

**STATISTICS DASHBOARDS –**

This is the real first step managing data statistics using a DCD (data
collection device) evolving toward a true analytics platform. In this
guide, we will explore setting up and establishing connectivity using
master key to each DCD (data collection device).

-  Enabling statistics for each functional area as part of the discovery
   process. This will allow BIG-IQ to proxy statistics gathered and
   organized from each BIG-IP device leveraging F5 Analytics iApp
   service (https://devcentral.f5.com/codeshare/f5-analytics-iapp).

-  Configuration and tuning of statistic collections post discovery
   allowing the user to focus on data specific to their needs.

-  Viewing and interaction with statistics dashboard, such as filtering
   views, differing time spans, selection and drilldown into dashboards
   for granular data trends and setting a refresh interval for
   collections.

**Auto-scaling in a VMware cloud environment**

You can now securely manage traffic to applications in a VMware cloud
environment, specifying the parameters in a service scaling group to
dynamically deploy and delete BIG-IP devices as needed. BIG-IQ manages
the BIG-IP devices that are load balancing to the BIG-IP VE devices in
the cloud, as well as to the BIG-IP devices' application servers.

**Auto-scaling in an AWS environment**

You can now securely manage traffic to applications in a VMware cloud
environment, specifying the parameters in a service scaling group to
dynamically deploy and delete BIG-IP devices as needed. You can manage
the BIG-IP VE devices from a BIG-IQ system on-premises, or in the cloud.
You have the option to use an F5 AWS Marketplace license, or your own
BIG-IP license.

**BIG-IQ VE deployment in MS Azure**

You can now deploy a BIG-IQ VE in a MS Azure cloud environment.

**Intuitive visibility for all managed applications**

BIG-IQ now provides an overview of all managed applications with the
option for a more detailed view of each application. Both the overview
and detailed views provide information about the application's
performance, Web Application Security status, and network statistics.

**Easy application troubleshooting based on application traffic and
security data**

You can now enable enhanced analytics to view detailed application data
in real-time, which allows you to isolate traffic characteristics that
are affecting your application's performance and security status.

**Real-time notifications for monitored devices and applications**

You can now receive real time alerts and events for BIG-IP devices and
their connected applications. These notifications are integrated into
the BIG-IQ UI charts and allow you to pinpoint activities that are
currently affecting your application.

**Enhanced HTTP and Web Application Security visibility for all
applications**

You can use the HTTP and Web Application Security Dashboards to monitor
all applications managed by BIG-IQ Centralized Management. These
dashboards allow you to compare applications, pool members, and other
aspects of traffic to your applications. In addition, the enhanced view
includes real time events and alerts within the charts, and enhanced
analytics data.

**Added object and management support for DNS features**

Creating, reading, updating, and deleting DNS GSLB objects, and
listeners is now supported from the BIG-IQ user interface and the API.

**Visibility into managed service scaling groups**

An automatically scalable environment of BIG-IP VE devices can be
defined to provide services to a set of applications. System
administrators of BIG-IQ Centralized Management can monitor performance
data for these BIG-IP VE devices.

**Enhanced DNS visibility & configuration**

BIG-IQ provides the ability to configure and have an enhanced view into
DNS traffic, which now includes both peak traffic values and average
traffic values over a selected period of time.

**Application templates**

Enhanced application/service templates that make deployments simple and
repeatable.

**Security policies and profiles available in applications**

You can now add security policies and profiles to applications,
including Web Application Security policies, Network Security firewall
policies, DoS profiles, and logging profiles.

**Automatically deploy policy learning**

You can now enable automatic deployment of policy learning using Web
Application Security.

**Extended ASM/advanced WAF management that includes**

-  Auto-deploy policy learning

-  Brute-force attack event monitoring

-  Event correlation

-  Manage DataSafe profiles

-  Initial ASM and HTTP monitoring dashboards

**Enhanced AFM Management**

-  AFM and DoS event visualization

-  Multi device packet tester

-  Enhanced debugging

**APM enhancements**

-  Management capabilities for APM Federation through BIG-IQ (SAML, IdP
   and SP)

-  Management capabilities for APM SSO configuration for Web Proxy
   Authentication Support Through BIG-IQ

**Manage cookie protection**

You can now manage cookie protection for BIG-IP devices using Web
Application Security.

**Monitoring dashboard for Web Application Security statistics**

You can review Web Application Security policy statistics using a
graphical dashboard.

**Manage DataSafe profiles**

You can now manage DataSafe profiles using Fraud Protection Security.

**Enhanced support for NAT firewalls**

You can now use the enhanced NAT firewall support in Network Security.

**Subscriber support in firewall rules**

You can now add subscriber IDs and groups to firewall rules in Network
Security for BIG-IP devices that support them.

**Firewall testing using packet flow reports**

You can now create and view packet flow reports to test firewall
configurations in Network Security.

**Support for multiple BIG-IP devices with packet tester reports**

You can now select multiple BIG-IP devices when generating packet tester
reports in Network Security.

**Renaming of firewall objects supported**

You can now rename firewall objects, such as firewall policies in
Network Security.

**Enhanced support for DoS profiles, device DoS configurations, and
scrubber profiles**

You can now manage additional features of DoS profiles, device DoS
configurations, and scrubber profiles that are found in BIG-IP version
13.1, such as new vectors, stress-based mitigation, DNS dynamic
signatures, and VLAN support in scrubber profiles.

**Copying device DoS configurations**

You can now copy device DoS configurations from one BIG-IP device to
multiple BIG-IP devices with the same version.

**Viewing logs for DoS and firewall events in the user interface**

You can now configure and view logging of DoS and firewall events, and
for DoS events, see that information in a graphical format.

Additional details can be found in the full release notes:

https://support.f5.com/kb/en-us/products/big-iq-centralized-mgmt/releasenotes/product/relnote-big-iq-central-mgmt-6-0-0.html

**BIG-IP Versions** AskF5 SOL with this info:

https://support.f5.com/kb/en-us/solutions/public/14000/500/sol14592.html

.. _section-7:

Changes to BIG-IQ User Interface
================================

The user interface in the 6.0 release navigation has changed to a more
UI tab-based framework.

In this section, we will go through the main features of the user
interface. Feel free to log into the BIG-IQ (https://192.168.1.50)
username: admin password: 401elliottW! device to explore some of these
features in the lab.

After you log into BIG-IQ, you will notice:

-  A navigation tab model at the top of the screen to display each high
   level functional area.

-  A tree based menu on the left-hand side of the screen to display
   low-level functional area for each tab.

-  A large object browsing and editing area on the right-hand side of
   the screen.

|image92|

-  Let us look a little deeper at the different options available in the
   bar at the top of the page.

|image93|

-  At the top, each tab describes a high-level functional area for
   BIG-IQ central management:

-  Monitoring –Visibility in dashboard format to monitor performance and
   isolate fault area.

-  Configuration – Provides configuration editors for each module area.

-  Deployment – Provides operational functions around deployment for
   each module area.

-  Devices – Lifecycle management around discovery, licensing and
   software install / upgrade.

-  System – Management and monitoring of BIG-IQ functionality.

-  Applications – Build, deploy, monitor service catalog-based
   applications centrally.

WORKFLOW 1: Creating a Backup Schedule

BIG-IQ is capable of centrally backing up and restoring all the BIG-IP
devices it manages. To create a simple backup schedule, follow the
following steps.

1. Click on the **Back Up & Restore** submenu in the Devices header.

2. |image94|\ Expand the **Back Up and Restore** menu item found on the
   left and click on **Backup Schedules**

3. Click the **Create** button

   |image95|

4. | Fill out the Backup Schedule using the following settings:
   | **Name:** Nightly
   | **Local Retention Policy:** Delete local backup copy 1 day after
     creation
   | **Backup Frequency:** Daily
   | **Start Time:** 00:00 Eastern Daylight Time
   | **Devices: Groups (radio button):** All BIG-IP Group Devices
   | Your screen should look similar to the one below.

|image96|\ **
**

5. Click **Save & Close** to save the scheduled backup job.

6. Optionally feel free to select the newly created schedule and select
   “Run Schedule Now” to immediately backup the devices.

   a. Add a Name for the Back Up

   b. Click **Start**

   c. When completed the backups will be listed under the Backup Files
      section

WORKFLOW 2: Uploading QKviews to iHealth for a support case
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BIG-IQ can now push qkviews from managed devices to ihealth.f5.com and
provide a link to the report of heuristic hits based on the qkview.
These qkview uploads can be performed ad-hoc or as part of a F5 support
case. If a support case is specified in the upload job, the qkview(s)
will automatically be associated/linked to the support case. In addition
to the link to the report, the qkview data is accessible at
ihealth.f5.com to take advantage of other iHealth features like the
upgrade advisor.

1. Navigate to **Monitoring** **Reports** **Device** **iHealth
   Configuration**

..

   |image97|

2. | |image98|\ Add Credentials to be used for the qkview upload and
     report retrieval.
   | \*If you do not have credentials, please raise your hand and speak
     to an instructor\* Click the Add button under Credentials.

3. | Fill in the credentials that you used to access
     https://ihealth.f5.com:
   | Name: Give the credentials a name to be referenced in BIG-IQ
   | Username: <Username you use to access iHealth.f5.com>
   | Password: <Password you use to access iHealth.f5.com>
   | |image99|

4. Click the Test button to validate that your credentials work.

5. Click the Save & Close button in the lower right.

6. Click the QKview Upload Schedules button in the BIG-IP iHealth menu.

..

   **Monitoring** **Reports** **Device** **iHealth** **QKView Upload
   Schedule**

7. Click Create with the following values

   a. Name – Weekly Upload

   b. Description – Nightly QKView Upload

   c. Credential – (use what was created in step 3)

   d. Upload Frequecny – Weekly (Select Sunday)

   e. Start Time – Select todays date at 00:00

   f. End Date – No End date should be checked

   g. Select both devices

   h. Click the right arrow to move to the “Selected” Area

8. Click Save & Close\ |image100|

You will now have a fresh set of QKView in iHealth every Sunday morning.
This is extremely useful for when new cases are opened, one less step
you’ll need for support to engage quicker.

WORKFLOW 3: Device Import

BIG-IQ is capable of centrally managing multiple products, for this lab
we will only manage LTM and AFM. To import the device configurations,
follow the steps below

1. Navigate to the Devices tab and click on **BIG-IP Devices** (left
   panel)

|image101|

2. You’ll notice both devices have not completed the import tasks, to
   remedy this simply click on the “Complete Import Tasks” Link

3. First Re-discover the LTM service

4. Then Discover the AFM service

5. Once Re-discovery has completed, import both the LTM and AFM services

6. Repeat this same procedure for both devices, once completed your
   screen will show the following.

   a. For any conflicts you may encounter – leave BIG-IQ selected
      resolution

|image102|

BIG-IQ Statistics Dashboards 
=============================

WORKFLOW 1: Reviewing the data in the dashboards
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Navigate to **Monitoring Dashboards Device Health**

|image103|

WORKFLOW 2: Interacting with the data in the dashboards
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  | You can narrow the scope of what is graphed by selecting a object
     or objects from the selection panels on the right. For example, if
     you only want to see data from BIG-IP01, you can click on it to
     filter the data.
   | |image104|

-  You can create complex filters by making additional selections in
   other panels

-  | You can zoom in on a time, by selecting a section of a graph or
     moving the slider at the top of the page
   | |image105|
   | or
   | |image106|

-  All the graphs update to the selected time.

-  | You can change how far in the data you want to look back by using
     the selection in the upper left (note you may need to let some time
     elapse before this option becomes available)
   | |image107|

|image108|

**F5 BIG-IQ Training**

**Advance Firewall Manager (AFM) Workflows**

**AFM Lab - 05**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

|http://cdn.ws.citrix.com/wp-content/uploads/2011/08/ICSA_Cert_WebFirewall_2C_200DPI_720x374.jpg|

Network Security (AFM) Management Workflows
===========================================

WORKFLOW 1: Managing AFM from BIG-IQ
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Day 4, it turns out no one thought about managing the new web and
application servers, as such SSH is blocked to both devices. Let’s first
validate this by using the packet tester tool within BIG-IQ, note this
is the same tool within BIG-IP with one major exception. Within BIG-IQ
you can trace a packet through **more than one firewall**. This is very
useful if you have multiple AFM devices in a packets path, now you can
test the flow end to end from one central location.

Task 1 – Packet Tracer
^^^^^^^^^^^^^^^^^^^^^^

Navigate to **Monitoring tab Reports Security Network Security Packet
Traces**

|image109|

Click on the “Create” button from the top menu.

Complete the following information

-  Name – ssh_trace

-  Protocol – tcp

-  TCP Flags – Syn

-  Source IP Address – 10.20.0.200

-  Source Port – 9999

-  Destination IP Address – 10.30.0.50

-  Destination Port – 22

-  Use Staged Policy – No

-  Trigger Log – No

Under the Devices section click “Add” (notice you’ll see all the devices
with AFM provision listed), for our lab however; just add
**bigip2.dnstest.lab**

|image110|

Select the “/Common/OUTSIDE” Vlan as the Source VLAN from the dropdown.

When completed your screen should look like the screen shot below:

|image111|

Click “Run Trace”

You can see from the trace results; the traffic is indeed being denied

|image112|

Another nice feature of Packet Trace within BIG-IQ is the ability to
clone a trace, when you complete the next two tasks, we’ll return to the
packet tracer tool to re-run the results using the clone option.
Additionally, the traces are saved and can be reviewed later, this can
be very helpful in long troubleshooting situations where application
teams are asking for results after changes are made to policies.

Follow the steps below to allow SSH access to both devices using BIG-IQ
as a central management tool.

**Task 2 – Modify Rule Lists**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Navigate to the **Configuration tab Security Network Security Rule
Lists**

Notice the previously created rule lists have been imported into BIG-IQ

Click on the “\ **application_rule_list**\ ”

Click **Create Rule** button.

Click on the pencil (edit rule) of the newly created rule listed with
**Id** of **2.**

Create a new rule with the below information.

Be prepared to scroll to find all the options

+-------------------------+-----------------------+
| **Name**                | allow_ssh             |
+=========================+=======================+
| **Source Address**      | **10.20.0.200**       |
+-------------------------+-----------------------+
| **Source Port**         | **any**               |
+-------------------------+-----------------------+
| **Source VLAN**         | **any**               |
+-------------------------+-----------------------+
| **Destination Address** | **10.30.0.50**        |
+-------------------------+-----------------------+
| **Destination Port**    | **22**                |
+-------------------------+-----------------------+
| **Action**              | **Accept-Decisively** |
+-------------------------+-----------------------+
| **Protocol**            | **TCP**               |
+-------------------------+-----------------------+
| **State**               | **enabled**           |
+-------------------------+-----------------------+
| **Log**                 | **True (checked)**    |
+-------------------------+-----------------------+

Click **Save & Close** when finished.

Repeat the same procedure for the web_rule_list, be sure to change the
destination to 10.30.0.50, all other setting remains the same.

**Task 3 – Deploy the Firewall Policy and related configuration objects**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _section-8:

**
**
^^

Now that the desired firewall configuration has been created on the
BIG-IQ, you need to deploy it to the BIG-IP. In this task, you create
the deployment, verify it, and deploy it.

From the top navigation bar, click on **Deployment** (tab).

Click on the **EVALUATE & DEPLOY** section on the left to expand it.

Click on **Network Security** in the expansion.

|image113|

Click on the top **Create** button under the **Evaluations** section.

Give your evaluation a name (ex: **deploy_afm1**).

Evaluation **Source** should be **Current Changes** (default).

Source Scope should be **All Changes** (default)

Remove Unused Objects should be **Remove Unused Objects** (default)

Target Device(s) should be **Device**.

Select **bigip2.dnstest.lab** from the list of Available devices and
move it to Selected area.

|image114|

Click the **Create** button at the bottom right of the page.

You should be redirected to the main **Evaluate and Deploy** page.

-  This will start the evaluation process in which BIG-IQ compares its
   working configuration to the configuration active on each BIG-IP.
   This can take a few moments to complete.

The **Status** section should be dynamically updating… (What states do
you see?)

Once the status shows **Evaluation Complete** you can view the
evaluation results.

-  Before selecting to deploy, feel free to select the differences
   indicated to see the proposed deployment changes. This is your check
   before making changes on a BIG-IP.

Click the number listed under **Differences – Firewall**.

Scroll through the list of changes to be deployed.

Click on a few to review in more detail.

What differences do you see from the **Deployed on BIG-IP** section and
on **BIG-IQ**?

Do you see the new rules you created in BIG-IQ? Ya should…

Click **Cancel**.

Deploy your changes by checking the box next to your evaluation
**deploy_afm1**.

With the box checked, click the **Deploy** button.

Your evaluation should move to the **Deployments** section.

After deploying, the status should change to **Deployment Complete**.

-  This will take a moment to complete. Once completed, log in to the
   BIG-IP and verify that the changes have been deployed to the AFM
   configuration.

Congratulations, you just deployed your first AFM policy via BIG-IQ!

Review the configuration deployed to the BIG-IP units.

On **bigip2.dnstest.lab**: (https://192.168.1.150)

Navigate to Security > Network Firewall > Policies.

Click on rd_0_policy and expand the rule lists

Are the two rules you created in BIG-IQ listed for this newly deployed
firewall policy?

|image115|

|image116|

Test Access
-----------

-  Open a new Web browser and access http://10.30.0.50

-  Open Putty and access 10.30.0.50

.. _section-9:

Task 4 – Packet Tracer (continued)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Navigate to the Monitoring tab Reports Security Network Security Packet
Tracers

Highlight the previous trace (ssh_trace) and click on the “Clone” button

|image117|

You’ll notice all the previously entered values are pre-populated, you
now can make any changes if necessary (maybe the application team
realized the source port of the flow is not random).

Click “Run Trace”

|image118|

SUCCESS!!

The history within the tool makes Root Cause Analysis (RCA) reports very
easy, this allows the security team to show a denied flow and subsequent
permitted flow.

WORKFLOW 2: Configure Network Security and DoS Event Logging 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Task 1 – Configure Network Security and DoS Event Logging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You enable Network Security event logging using the virtual servers
displayed in the context list

Navigate to the Configuration Security Network Security Contexts

Check the box next to the IPV4_TCP VIP

Select “Configure Logging” from the top buttons

|image119|

You will receive a configuration message alerting you to the changes
about to be made to the device, click Continue

|image120|

This will now configure a logging profile, associated pools, monitors
and all necessary configuration to send logs to the Data Collection
Device (DCD).

In the spirit of central management, we’re also going to configure the
DoS event logging, so we only must perform one deployment on both
devices.

Navigate to Configuration Security Shared Security DoS Protection Device
DoS Configurations

Highlight bigip1.dnstest.lab and click the “Configure DoS Logging”
button from the top.

|image121|

Once again you will receive a configuration message, click continue

Once completed navigate to the Deployments tab

As most of the configuration is “LTM” related you will first need to
deploy the LTM configuration.

Navigate to Evaluate & Deploy

Select Local Traffic & Network Traffic

Create an evaluation named “logging_configuration”, leave all other
defaults and select both devices, once finished, create the evaluation.

Feel free to examine the changes in the evaluation, when satisfied
deploy the changes.

Once the LTM configuration is deployed, you’ll need to also deploy the
Network Security portion of the changes.

Navigate to Deployment Evaluate & Deploy Network Security.

Again, create an evaluation and subsequent deployment for both devices.

Task 2 – Evaluate Network Firewall Events
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Browse to http://10.30.0.50 once again (or refresh in your tabs).

Within BIG-IQ, navigate to Monitoring Network Security Firewall

Click on a line item for enriched information in the window below as
shown

|image122|

Feel free to view other logs to see the data presented.

Task 3 – Evaluate DoS Events
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Open a few separate windows to the attack host. We will launch a few
attacks at once to see the value of consolidated reporting within BIG-IQ
(there is a text document on the jumbox desktop which contains all of
the attack commands).

Launch a few attacks at once and navigate to Monitoring Events –DoS DoS
Summary

|image123|

From here you have a consolidated view of all your devices and attacks.

Click on one of the attack ID’s for enriched information about the
attack

|image124|

This concludes the lab. You have had quite the eventful first week at
Initech! You have successfully allowed communication to a new webserver,
you tuned and defended against several DoS attacks, you then configured
BIG-IQ for central device management and monitoring and lastly, you’re
now managing AFM within BIG-IQ. I think you deserve Friday off!!

|image125|

**F5 BIG-IQ Training**

**Advance Firewall Manager – iControl REST API**

**AFM Lab - 06**

**Written for TMOS 13.1.0.1/BIG-IQ 6.0**

June 8, 2018

|http://cdn.ws.citrix.com/wp-content/uploads/2011/08/ICSA_Cert_WebFirewall_2C_200DPI_720x374.jpg|

It’s Friday, you’ve made it through week one, but its not over yet.
After another meeting with the Bob’s they’ve decided they want to
explore the SecOps world and configure devices through the REST API.
Before we proceed let’s learn a little about what REST is and how to
interact with the F5 API, also known as iControl.

**About Representational State Transfer**

Representational State Transfer (REST) describes an architectural style
of web services where clients and servers exchange representations of
resources. The REST model defines a resource as a source of information
and defines a representation as the data that describes the state of a
resource. REST web services use the HTTP protocol to communicate between
a client and a server, specifically by means of the POST, GET, PUT, and
DELETE methods to create, read, update, and delete elements or
collections. In general terms, REST queries resources for the
configuration objects of a BIG-IP® system, and creates, deletes, or
modifies the representations of those configuration objects. The
iControl® REST implementation follows the REST model by:

• Using REST as a resource-based interface, and creating API methods
based on nouns.

   • Employing a stateless protocol and MIME data types, as well as
   taking advantage of the authentication mechanisms and caching built
   into the HTTP protocol.

• Supporting the JSON format for document encoding.

   • Representing the hierarchy of resources and collections with a
   Uniform Resource Identifier (URI) structure.

   • Returning HTTP response codes to indicate success or failure of an
   operation.

• Including links in resource references to accommodate discovery.

**About URI format**

The iControl® REST API enables the management of a BIG-IP® device by
using web service requests. A principle of the REST architecture
describes the identification of a resource by means of a Uniform
Resource Identifier (URI). You can specify a URI with a web service
request to create, read, update, or delete some component or module of a
BIG-IP system configuration. In the context of REST architecture, the
system configuration is the representation of a resource. A URI
identifies the name of a web resource; in this case, the URI also
represents the tree structure of modules and components in TMSH.

In iControl REST, the URI structure for all requests includes the string
/mgmt/tm/ to identify the namespace for traffic management. Any
identifiers that follow the endpoint are resource collections.

Tip: Use the default administrative account, admin, for requests to
iControl REST. Once you are familiar with the API, you can create user
accounts for iControl REST users with various permissions.

https://management-ip/mgmt/tm/module

The URI in the previous example designates all of the TMSH subordinate
modules and components in the specified module. iControl REST refers to
this entity as an organizing collection. An organizing collection
contains links to other resources. The management-ip component of the
URI is the fully qualified domain name (FQDN) or IP address of a BIG-IP
device.

Important: iControl REST only supports secure access through HTTPS, so
you must include credentials with each REST call. Use the same
credentials you use for the BIG-IP device manager interface.

For example, use the following URI to access all the components and
subordinate modules in the LTM module:

https://management-ip/mgmt/tm/\ ltm

The URI in the following example designates all of the subordinate
modules and components in the specified sub-module. iControl REST refers
to this entity as a collection; a collection contains resources.

https://management-ip/mgmt/tm/module/sub-module

The URI in the following example designates the details of the specified
component. The Traffic Management Shell (TMSH) Reference documents the
hierarchy of modules and components, and identifies details of each
component. iControl REST refers to this entity as a resource. A resource
may contain links to sub-collections.

`https://management-ip/mgmt/tm/module[/sub-module]/component <https://management-ip/mgmt/tm/module%5b/sub-module%5d/component>`__

**About reserved ASCII characters**

To accommodate the BIG-IP® configuration objects that use characters,
which are not part of the unreserved ASCII character set, use a percent
sign (%) and two hexadecimal digits to represent them in a URI. The
unreserved character set consists of: [A - Z] [a - z] [0 - 9] dash (-),
underscore (_), period (.), and tilde (~).

You must encode any characters that are not part of the unreserved
character set for inclusion in a URI scheme. For example, an IP address
in a non-default route domain that contains a percent sign to indicate
an address in a specific route domain, such as 192.168.25.90%3, should
be encoded to replace the %character with %25.

**About REST resource identifiers**

A URI is the representation of a resource that consists of a protocol,
an address, and a path structure to identify a resource and optional
query parameters. Because the representation of folder and partition
names in TMSH often includes a forward slash (/), URI encoding of folder
and partition names must use a different character to represent a
forward slash in iControl®

To accommodate the forward slash in a resource name, iControl REST maps
the forward slash to a tilde (~) character. When a resource name
includes a forward slash (/) in its name, substitute a tilde (~) for the
forward slash in the path. For example, a resource name, such as
/Common/plist1, should be modified to the format shown here:

https://management-ip/mgmt/tm/security/firewall/port-list/~Common~plist1

**About Postman – REST Client**

Postman helps you be more efficient while working with APIs. Postman is
a scratch-your-own-itch project. The need for it arose while one of the
developers was creating an API for his project. After looking around for
a number of tools, nothing felt just right. The primary features added
initially were a history of sent requests and collections. You can find
Postman here - `www.getpostman.com <http://www.getpostman.com>`__.

Simulating and defeating a Christmas Tree Packet Attack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we understand what REST is let’s use it to defeat Joanna one
last time. Joanna was feeling festive for her final attack. In this
example, we’ll set the BIG-IP to detect and mitigate Joanna’s attack
where all flags on a TCP packet are set. This is commonly referred to as
a Christmas tree packet and is intended to increase processing on
in-path network devices and end hosts to the target.

To interact with the REST API, we’ll be using POSTMan. We’ll then use
the hping utility to send 25,000 packets to our server, with random
source IPs to simulate a DDoS attack where multiple hosts are attacking
our server. We’ll set the SYN, ACK, FIN, RST, URG, PUSH, Xmas and Ymas
TCP flags.

13. POSTMan is installed as an application and can be accessed from the
    desktop of the Jumpbox

14. Once you launch POSTMan You’ll then want to import the API calls for
    the lab as well as the environment variables

    f. There is a notepad on the desktop labeled “Postman Links”

    g. Within POSTman and click on the “Import” link near the top and
       then select “Import from Link”

    h. Copy and paste the **collection** link from within the notepad
       and select “Import”

    i. Copy and paste the **environment** link from within the notepad
       and select “Import”

|image126|

15. Before proceeding verify the Agility 2018 environment is selected
    from the drop down in the top right of POSTman

|image127|

16. In the bigip01.dnstest.lab (https://192.168.1.100) web UI, navigate
    to Security > DoS Protection > Device Configuration > Network
    Security.

17. Expand the **Bad-Header-TCP** category in the vectors list.

18. Click on the **Bad TCP Flags (All Flags Set)** vector name and take
    note of the current settings

19. Within POSTman open the collection “Agility 2018 Lab 5”

    |image128|

20. Run step 1 by clicking on the send button to the right

|image129|

21. The output from the GET request can be reviewed, this is showing you
    all the device-dos configuration options and settings. Search for
    "bad-tcp-flags-all-set” by clicking ‘ctrl +f’. Note the values as
    they are currently configured. We are now going to modify the Bad
    TCP Flags (All Flags Set) attack vector. To do so run step 2 of the
    collection by highlighting the collection and click “Send”.

22. You can now execute step 3 in the collection and verify the changes,
    you can also verify the changes in the BIG-IP web UI.

|image130|

23. Open the BIG-IP SSH session and scroll the ltm log in real time with
    the following command: tail -f /var/log/ltm

24. | On the attack host, launch the attack by issuing the following
      command on the BASH prompt:
    | sudo hping3 10.20.0.10 --flood --rand-source --destport 80 -c
      25000 --syn --ack --fin --rst --push --urg --xmas --ymas

25. | You’ll see the BIG-IP ltm log show that the attack has been
      detected:
    | |image131|

26. | After approximately 60 seconds, press **CTRL+C** to stop the
      attack.
    | |image132|

27. Navigate to **Security** > **DoS Protection**> **DoS Overview (you
    may need to refresh or set the auto refresh to 10 seconds).** You’ll
    notice from here you can see all the details of the active attacks.
    You can also modify an attack vector right from this screen by
    clicking on the attack vector and modifying the fly out. <- did you
    mean to say “on the fly” JM

    |image133|

28. | Return to the BIG-IP web UI. Navigate to **Security** > **Event
      Logs** > **DoS** > **Network** > **Events**. Observe the log
      entries showing the details surrounding the attack detection and
      mitigation.
    | |image134|

29. Navigate to **Security** > **Reporting** > **DoS** > **Analysis**.
    Single-click on the attack ID in the filter list to the right of the
    charts and observe the various statistics around the attack.

30. The same attacks can also be seen in BIG-IQ as demonstrated in the
    previous lab.

Congratulations, you have successfully defeated Joanna’s festive attack
using only the REST API to configure the device! Since it’s the end of
the week and Joanna is using the same IP address continually, lets block
her IP address and her subnet using BIG-IQ. We’ll use the REST API to
accomplish this as well, as BIG-IQ also has an available REST API.

1. Using POSTman run step 4, this will create an address-list within
   BIG-IQ, the advantage to address-lists is they allow you to group
   similar objects into a group. In this instance we’re going to create
   an address-list named API_Naughty_Address_List with a host and a
   network. Once you run the command you’ll receive output below. You
   will need to copy the value returned in the ‘ID” field as shown
   below:

|image135|

2. Take the copied text and paste it into the environment variable for
   AFM_Adddress_ID. The variables are accessed by clicking on the “eye”
   icon next to where you selected the Agility 2018 Environment:

..

   |image136|

3. Click edit and enter the value returned in step 1, when completed
   click update

|image137|

4. We will now create a rule list name first, to accomplish this send
   the call found in step 5. You will need to also capture the “ID” in
   this step as well. This value will be updated in the AFM_Rule_ID
   field

|image138|

5. Take the copied text and paste it into the environment variable for
   AFM_Rule_ID

|image139|

6. At this stage we have created an address-list with objects and saved
   the ID, we have also created a rule name and saved the ID. The next
   step is to add an actual rule to the newly created rule named
   “Naughty_Rule_List”. Before you send the call-in step 6, take a
   moment to examine the body of the request. You’ll notice in the URI
   we’re referencing the variable of AFM_Rule_ID and in the body of the
   JSON request we’re linking the AFM_Address_ID to the rule. Once sent
   you’ll receive confirmation similar to the below output.

|image140|

7. Since this is an existing environment, we’re going to first need to
   obtain the policy ID before we can assign the value to this variable.
   To obtain the policy ID of the existing policy we created in lab 1
   and imported in the prior lab, run step 7.

|image141|

8. You will notice there are two policies, Global and rd_0_policy, we’ll
   need to copy the ID for the rd_0_policy which is located directly
   under its name and paste it into the variable for AFM_Policy_ID.

|image142|

9. Finally run step 8 to add the new rule list to the existing policy,
   when completed you’ll receive output similar as seen below.

|image143|

10. Before we deploy the policy. Log into the BIG-IQ web UI
    (https://192.168.1.50) and navigate to Configuration Security
    Network Security Firewall Policies. Click on the link for the
    rd_0_policy, expand all the rules to verify your new API created
    rule list is first in the list and all objects are created as
    expected.

|image144|

11. The final step is to deploy the policy to the BIG-IP. Before we can
    do this, we have one last variable we’ll need to acquire, the
    machine ID of bigip02.dnslab.test. To obtain the machine ID run the
    call in step 9, once the call is run, you will look for the
    machineId key and copy the value to the environment variable
    bigip02-machined as shown below and click update.

|image145|

|image146|

12. Finally, you will run step 10, this will initiate a deployment on
    BIG-IQ to deploy the changes to BIG-IP. Within BIG-IQ navigate to
    Deployment Evaluate & Deploy Network Security. At the bottom in the
    deployments section you’ll notice an API Policy Deploy task. Feel
    free to click on the task to investigate the changes. Once the
    policy has deployed, log into the web UI of bigip02.dnstest.lab and
    navigate to Security network Firewall Active Rules. Change the
    context to Route Domain and select 0. Expand all of the rules to
    verify the rules have been deployed as expected. Your final screen
    should look something like the screen capture below.

|image147|

Lastly, in your web browser verify you can no longer access the web
pages http://10.30.0.50 and http://10.40.0.50 as well as no longer being
able to SSH to any of the devices.

Congratulations, you have made it through your first week at Initiech.
Before you leave for the weekend please be sure to update your TPS
report with the weeks findings

Thank you very much for attending the lab, we hope you enjoyed the
content!

.. |image0| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |https://www.icsalabs.com/sites/default/files/imagecache/large_logo/ICSA_Cert_Firewall_WEB.gif| image:: .//media/image2.gif
   :width: 1.82083in
   :height: 1.23889in
.. |image2| image:: .//media/image3.png
   :width: 6.49097in
   :height: 3.23125in
.. |image3| image:: .//media/image4.png
   :width: 3.54122in
   :height: 2.67675in
.. |image4| image:: .//media/image5.png
   :width: 6.5in
   :height: 1.87222in
.. |http://support.f5.com/kb/global/manual_images/MAN-0439-01/firewall_processing.png| image:: .//media/image6.png
   :width: 4.71829in
   :height: 5.18219in
.. |image6| image:: .//media/image7.png
   :width: 6.49097in
   :height: 1in
.. |image7| image:: .//media/image8.png
   :width: 6.5in
   :height: 1.46319in
.. |image8| image:: .//media/image9.png
   :width: 6.5in
   :height: 0.80278in
.. |image9| image:: .//media/image10.png
   :width: 3.25in
   :height: 1.46554in
.. |image10| image:: .//media/image11.png
   :width: 6.2954in
   :height: 1.66667in
.. |image11| image:: .//media/image12.png
   :width: 5.87014in
   :height: 6.17122in
.. |image12| image:: .//media/image13.png
   :width: 6.49097in
   :height: 0.25903in
.. |image13| image:: .//media/image14.png
   :width: 4.92847in
   :height: 1.35694in
.. |image14| image:: .//media/image15.png
   :width: 6.5in
   :height: 0.60208in
.. |image15| image:: .//media/image16.png
   :width: 6.5in
   :height: 1.29653in
.. |image16| image:: .//media/image17.png
   :width: 6.49097in
   :height: 0.47222in
.. |image17| image:: .//media/image18.png
   :width: 6.5in
   :height: 1in
.. |image18| image:: .//media/image19.png
   :width: 6.5in
   :height: 1.25in
.. |image19| image:: .//media/image20.png
   :width: 6.49514in
   :height: 1.37014in
.. |image20| image:: .//media/image21.png
   :width: 6.49097in
   :height: 0.70347in
.. |image21| image:: .//media/image22.png
   :width: 6.48125in
   :height: 0.60208in
.. |image22| image:: .//media/image23.png
   :width: 6.49097in
   :height: 1.76875in
.. |image23| image:: .//media/image24.png
   :width: 2.67327in
   :height: 2.26704in
.. |image24| image:: .//media/image25.png
   :width: 6.5in
   :height: 1.25903in
.. |image25| image:: .//media/image26.png
   :width: 6.5in
   :height: 1.22222in
.. |image26| image:: .//media/image27.png
   :width: 6.49097in
   :height: 0.75in
.. |image27| image:: .//media/image28.png
   :width: 6.5in
   :height: 1.66667in
.. |image28| image:: .//media/image29.png
   :width: 6.5in
   :height: 1.02778in
.. |image29| image:: .//media/image30.png
   :width: 6.49097in
   :height: 1.01875in
.. |image30| image:: .//media/image31.png
   :width: 6.5in
   :height: 1.14792in
.. |image31| image:: .//media/image32.png
   :width: 6.5in
   :height: 0.5in
.. |image32| image:: .//media/image32.png
   :width: 6.5in
   :height: 0.5in
.. |image33| image:: .//media/image33.png
   :width: 6.5in
.. |image34| image:: .//media/image34.png
   :width: 6.49097in
   :height: 0.59236in
.. |image35| image:: .//media/image35.png
   :width: 6.49097in
   :height: 2.49097in
.. |image36| image:: .//media/image36.png
   :width: 6.5in
   :height: 2.5in
.. |image37| image:: .//media/image37.png
   :width: 2.64727in
   :height: 1.53731in
.. |image38| image:: .//media/image38.png
   :width: 6.5in
   :height: 2.66667in
.. |image39| image:: .//media/image39.png
   :width: 6.5in
   :height: 2.08333in
.. |image40| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |image41| image:: .//media/image40.png
   :width: 6.5in
   :height: 3.44792in
.. |image42| image:: .//media/image41.png
   :width: 6.48958in
   :height: 3.44792in
.. |image43| image:: .//media/image42.png
   :width: 6.47361in
   :height: 2.94722in
.. |image44| image:: .//media/image43.png
   :width: 6.5in
   :height: 2.66667in
.. |image45| image:: .//media/image44.png
   :width: 6.49722in
   :height: 2.97708in
.. |image46| image:: .//media/image45.png
   :width: 6.48542in
   :height: 1.34653in
.. |image47| image:: .//media/image46.png
   :width: 6.49167in
   :height: 0.68819in
.. |image48| image:: .//media/image47.png
   :width: 3.55556in
   :height: 3.70347in
.. |image49| image:: .//media/image48.png
   :width: 6.49722in
   :height: 1in
.. |image50| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |image51| image:: .//media/image49.png
   :width: 3.94702in
   :height: 3.80739in
.. |image52| image:: .//media/image50.png
   :width: 4.75828in
   :height: 4.42937in
.. |image53| image:: .//media/image51.png
   :width: 4.72535in
   :height: 3.47384in
.. |image54| image:: .//media/image52.png
   :width: 3.80731in
   :height: 8.22517in
.. |image55| image:: .//media/image53.png
   :width: 6.16667in
   :height: 0.44444in
.. |image56| image:: .//media/image54.png
   :width: 4.73472in
   :height: 6.057in
.. |image57| image:: .//media/image55.png
   :width: 5.6in
   :height: 3.53333in
.. |image58| image:: .//media/image56.png
   :width: 4.045in
   :height: 5.58775in
.. |image59| image:: .//media/image57.png
   :width: 4.94444in
   :height: 2.29167in
.. |image60| image:: .//media/image58.png
   :width: 3.54305in
   :height: 4.68726in
.. |image61| image:: .//media/image59.tiff
   :width: 6.5in
   :height: 2.78542in
.. |image62| image:: .//media/image60.png
   :width: 2.67129in
   :height: 5.81457in
.. |image63| image:: .//media/image61.png
   :width: 6.22083in
   :height: 3.675in
.. |image64| image:: .//media/image62.png
   :width: 3.81074in
   :height: 5.31788in
.. |image65| image:: .//media/image63.png
   :width: 4.17881in
   :height: 2.18072in
.. |image66| image:: .//media/image64.png
   :width: 3.40297in
   :height: 3.22001in
.. |image67| image:: .//media/image65.png
   :width: 2.96026in
   :height: 1.54149in
.. |image68| image:: .//media/image66.tiff
   :width: 2.36111in
   :height: 0.68056in
.. |image69| image:: .//media/image67.tiff
   :width: 4.875in
   :height: 1.68056in
.. |image70| image:: .//media/image68.png
   :width: 6.69121in
   :height: 3.10185in
.. |image71| image:: .//media/image69.tiff
   :width: 4.3245in
   :height: 3.69594in
.. |image72| image:: .//media/image70.tiff
   :width: 5in
   :height: 4.06944in
.. |image73| image:: .//media/image71.png
   :width: 6.49097in
   :height: 2.17569in
.. |image74| image:: .//media/image72.png
   :width: 3.89968in
   :height: 3.43639in
.. |image75| image:: .//media/image73.png
   :width: 2.60017in
   :height: 6.93378in
.. |image76| image:: .//media/image74.png
   :width: 3.02244in
   :height: 2.63576in
.. |image77| image:: .//media/image75.png
   :width: 3.19205in
   :height: 5.75689in
.. |image78| image:: .//media/image76.tiff
   :width: 3.80132in
   :height: 1.11928in
.. |image79| image:: .//media/image77.png
   :width: 2.98013in
   :height: 1.62914in
.. |image80| image:: .//media/image78.png
   :width: 2.43392in
   :height: 2.49669in
.. |image81| image:: .//media/image79.tiff
   :width: 4.48611in
   :height: 0.38889in
.. |image82| image:: .//media/image80.tiff
   :width: 4.43056in
   :height: 0.97222in
.. |image83| image:: .//media/image81.png
   :width: 6.49097in
   :height: 1.10208in
.. |image84| image:: .//media/image82.tiff
   :width: 5in
   :height: 1.70833in
.. |image85| image:: .//media/image83.png
   :width: 2.45542in
   :height: 3.49669in
.. |image86| image:: .//media/image84.png
   :width: 2.7954in
   :height: 6.25166in
.. |image87| image:: .//media/image85.png
   :width: 2.56954in
   :height: 0.91286in
.. |image88| image:: .//media/image86.png
   :width: 6.49097in
   :height: 3.32431in
.. |image89| image:: .//media/image87.png
   :width: 2.41144in
   :height: 5.43046in
.. |image90| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |http://cdn.ws.citrix.com/wp-content/uploads/2011/08/ICSA_Cert_WebFirewall_2C_200DPI_720x374.jpg| image:: .//media/image88.jpeg
   :width: 1.94783in
   :height: 1.01123in
.. |image92| image:: .//media/image89.png
   :width: 6.5in
   :height: 0.91667in
.. |image93| image:: .//media/image90.png
   :width: 6.49097in
   :height: 2.67569in
.. |image94| image:: .//media/image91.png
   :width: 2.28056in
   :height: 1.23889in
.. |image95| image:: .//media/image92.png
   :width: 2in
   :height: 1.47917in
.. |image96| image:: .//media/image93.png
   :width: 6.49097in
   :height: 2.98125in
.. |image97| image:: .//media/image94.png
   :width: 6.5in
   :height: 2.7in
.. |image98| image:: .//media/image95.png
   :width: 1.88472in
   :height: 0.92639in
.. |image99| image:: .//media/image96.png
   :width: 3.37624in
   :height: 2.14141in
.. |image100| image:: .//media/image97.png
   :width: 5.13861in
   :height: 2.70482in
.. |image101| image:: .//media/image98.png
   :width: 6.5in
   :height: 1.12986in
.. |image102| image:: .//media/image99.png
   :width: 6.5in
   :height: 1.16667in
.. |image103| image:: .//media/image100.png
   :width: 6.5in
   :height: 2.34236in
.. |image104| image:: .//media/image101.png
   :width: 2.04653in
   :height: 1.5in
.. |image105| image:: .//media/image102.png
   :width: 4.91605in
   :height: 1.91643in
.. |image106| image:: .//media/image103.png
   :width: 4.48902in
   :height: 0.91655in
.. |image107| image:: .//media/image104.png
   :width: 2.40595in
   :height: 2.81215in
.. |image108| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |image109| image:: .//media/image105.png
   :width: 4.54514in
   :height: 4.10903in
.. |image110| image:: .//media/image106.png
   :width: 6.49097in
   :height: 3.76389in
.. |image111| image:: .//media/image107.png
   :width: 5.91089in
   :height: 3.67852in
.. |image112| image:: .//media/image108.png
   :width: 6.5in
   :height: 3.18194in
.. |image113| image:: .//media/image109.png
   :width: 4.65486in
   :height: 3.92708in
.. |image114| image:: .//media/image110.png
   :width: 6.5in
   :height: 2.40903in
.. |image115| image:: .//media/image111.png
   :width: 6.48194in
   :height: 1.71806in
.. |image116| image:: .//media/image112.png
   :width: 4.34028in
   :height: 2.38194in
.. |image117| image:: .//media/image113.png
   :width: 6.49097in
   :height: 0.85486in
.. |image118| image:: .//media/image114.png
   :width: 6.49097in
   :height: 3.13611in
.. |image119| image:: .//media/image115.png
   :width: 6.49097in
   :height: 2.19097in
.. |image120| image:: .//media/image116.png
   :width: 5.24514in
   :height: 2.43611in
.. |image121| image:: .//media/image117.png
   :width: 6.5in
   :height: 3.25486in
.. |image122| image:: .//media/image118.png
   :width: 6.5in
   :height: 2.59097in
.. |image123| image:: .//media/image119.png
   :width: 6.49097in
   :height: 2.7in
.. |image124| image:: .//media/image120.png
   :width: 6.5in
   :height: 3.6in
.. |image125| image:: .//media/image1.jpeg
   :width: 3.27778in
   :height: 1.14444in
.. |image126| image:: .//media/image121.png
   :width: 6.5in
   :height: 4.08333in
.. |image127| image:: .//media/image122.png
   :width: 3.86458in
   :height: 2.13542in
.. |image128| image:: .//media/image123.png
   :width: 2.50278in
   :height: 4.31042in
.. |image129| image:: .//media/image124.png
   :width: 6.49722in
   :height: 0.83056in
.. |image130| image:: .//media/image78.png
   :width: 2.43392in
   :height: 2.49669in
.. |image131| image:: .//media/image79.tiff
   :width: 4.48611in
   :height: 0.38889in
.. |image132| image:: .//media/image80.tiff
   :width: 4.43056in
   :height: 0.97222in
.. |image133| image:: .//media/image81.png
   :width: 6.49097in
   :height: 1.10208in
.. |image134| image:: .//media/image82.tiff
   :width: 5in
   :height: 1.70833in
.. |image135| image:: .//media/image125.png
   :width: 6.49097in
   :height: 1.82708in
.. |image136| image:: .//media/image126.png
   :width: 3.81806in
   :height: 3.66389in
.. |image137| image:: .//media/image127.png
   :width: 5.99097in
   :height: 6.22708in
.. |image138| image:: .//media/image128.png
   :width: 6.49097in
   :height: 1.31806in
.. |image139| image:: .//media/image129.png
   :width: 5.98194in
   :height: 6.18194in
.. |image140| image:: .//media/image130.png
   :width: 6.5in
   :height: 4.83333in
.. |image141| image:: .//media/image131.png
   :width: 6.49097in
   :height: 3.15486in
.. |image142| image:: .//media/image132.png
   :width: 6in
   :height: 3.33333in
.. |image143| image:: .//media/image133.png
   :width: 6.49097in
   :height: 1.87292in
.. |image144| image:: .//media/image134.png
   :width: 6.5in
   :height: 1.69097in
.. |image145| image:: .//media/image135.png
   :width: 6.5in
   :height: 1.27292in
.. |image146| image:: .//media/image136.png
   :width: 6in
   :height: 3.33333in
.. |image147| image:: .//media/image137.png
   :width: 6.5in
   :height: 3.23611in

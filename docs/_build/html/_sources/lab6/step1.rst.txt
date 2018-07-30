Lab Overview
============

It’s Friday, you’ve made it through week one, but its not over yet.
After another meeting with the Bob’s they’ve decided they want to
explore the SecOps world and configure devices through the REST API.
Before we proceed let’s learn a little about what REST is and how to
interact with the F5 API, also known as iControl.

About Representational State Transfer
=====================================

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

- Using REST as a resource-based interface, and creating API methods based on nouns.

  - Employing a stateless protocol and MIME data types, as well as taking advantage of the authentication mechanisms and caching built into the HTTP protocol.

- Supporting the JSON format for document encoding.

  - Representing the hierarchy of resources and collections with a Uniform Resource Identifier (URI) structure.
  - Returning HTTP response codes to indicate success or failure of an operation.

- Including links in resource references to accommodate discovery.

About URI format
================

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

https://management-ip/mgmt/tm/ltm

The URI in the following example designates all of the subordinate
modules and components in the specified sub-module. iControl REST refers
to this entity as a collection; a collection contains resources.

https://management-ip/mgmt/tm/module/sub-module

The URI in the following example designates the details of the specified
component. The Traffic Management Shell (TMSH) Reference documents the
hierarchy of modules and components, and identifies details of each
component. iControl REST refers to this entity as a resource. A resource
may contain links to sub-collections.

https://management-ip/mgmt/tm/module/[sub-module]/component

About reserved ASCII characters
===============================

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

About REST resource identifiers
===============================

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

About Postman – REST Client
===========================

Postman helps you be more efficient while working with APIs. Postman is
a scratch-your-own-itch project. The need for it arose while one of the
developers was creating an API for his project. After looking around for
a number of tools, nothing felt just right. The primary features added
initially were a history of sent requests and collections. You can find
Postman here - `www.getpostman.com <http://www.getpostman.com>`__.

Simulating and defeating a Christmas Tree Packet Attack
=======================================================

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

.. |image126| image:: ../media/image121.png
   :width: 6.5in
   :height: 4.08333in
.. |image127| image:: ../media/image122.png
   :width: 3.86458in
   :height: 2.13542in
.. |image128| image:: ../media/image123.png
   :width: 2.50278in
   :height: 4.31042in
.. |image129| image:: ../media/image124.png
   :width: 6.49722in
   :height: 0.83056in
.. |image130| image:: ../media/image78.png
   :width: 2.43392in
   :height: 2.49669in
.. |image131| image:: ../media/image79.tiff
   :width: 4.48611in
   :height: 0.38889in
.. |image132| image:: ../media/image80.tiff
   :width: 4.43056in
   :height: 0.97222in
.. |image133| image:: ../media/image81.png
   :width: 6.49097in
   :height: 1.10208in
.. |image134| image:: ../media/image82.tiff
   :width: 5in
   :height: 1.70833in
.. |image135| image:: ../media/image125.png
   :width: 6.49097in
   :height: 1.82708in
.. |image136| image:: ../media/image126.png
   :width: 3.81806in
   :height: 3.66389in
.. |image137| image:: ../media/image127.png
   :width: 5.99097in
   :height: 6.22708in
.. |image138| image:: ../media/image128.png
   :width: 6.49097in
   :height: 1.31806in
.. |image139| image:: ../media/image129.png
   :width: 5.98194in
   :height: 6.18194in
.. |image140| image:: ../media/image130.png
   :width: 6.5in
   :height: 4.83333in
.. |image141| image:: ../media/image131.png
   :width: 6.49097in
   :height: 3.15486in
.. |image142| image:: ../media/image132.png
   :width: 6in
   :height: 3.33333in
.. |image143| image:: ../media/image133.png
   :width: 6.49097in
   :height: 1.87292in
.. |image144| image:: ../media/image134.png
   :width: 6.5in
   :height: 1.69097in
.. |image145| image:: ../media/image135.png
   :width: 6.5in
   :height: 1.27292in
.. |image146| image:: ../media/image136.png
   :width: 6in
   :height: 3.33333in
.. |image147| image:: ../media/image137.png
   :width: 6.5in
   :height: 3.23611in
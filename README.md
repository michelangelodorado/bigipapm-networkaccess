# bigipapm-networkaccess

Manual: BIG-IP Access Policy Manager: Network Access
https://techdocs.f5.com/en-us/bigip-16-1-0/big-ip-access-policy-manager-network-access.html

The BIG-IP Access Policy Manager network access feature provides secure access to corporate applications and data using a standard web browser, or the BIG-IP Edge Client. Using network access, employees, partners, and customers can have access to corporate resources securely, from any location. 

The network access feature provides users with the functionality of a traditional IPsec VPN client. Unlike IPsec, however, network access does not require any pre-installed software or configuration on the remote user's computer. It is also more robust than IPsec VPN against router and firewall incompatibilities.

<img width="468" alt="image" src="https://github.com/michelangelodorado/bigipapm-networkaccess/assets/102953584/f88cafd6-4d29-4d58-8b19-056dcb30d16e">

Use this above dependency diagram and steps below to create and review the objects required to manually configure a working Network Access configuration.  Many objects like Connectivity Profiles can be reused. 

Objects in RED on the dependency diagram are only required when Data Transport Layer Security (DTLS) is enabled or the HTTP redirect option is desired.  The HTTP redirect is just a simple HTTP VS which has a system iRule redirecting users to HTTPS so users do not have to enter HTTPS in their browser to connect and can simply type an FQDN.  Objects in RED italic are not mandatory for Network Access to function, but are recommended.  Please complete them for this lab.

Begin in the section for Network Access in Access >> Connectivity / VPN.  All settings you configure below are selectable under the menu “Network Access (VPN)”.

1.	Create a new IPv4 lease pool

tmsh create apm resource leasepool <name> members add { 1.1.1.2-1.1.1.10 }
 
The range of IPs used should not exist anywhere else in the network because they will be assigned to users.
 
2.	Create Network Access object

tmsh create apm resource network-access <name> leasepool-name <name>

5.	You already have a AAA server for AD which can be reused and you don’t have to worry about NTP and DNS because they have already been configured.
6.	You already have a Full Webtop object available for use from the Portal Access lab.



11.	This created your new Network Access object (Similar to creating a Portal Access object.  A “container” for most of the settings, so don’t navigate away once you click Finished, you must set the remaining required settings or it will not function).  
12.	Click on the Network Settings tab. (if a setting is not mentioned leave it at its default setting!)
13.	Ensure Advanced settings are selected so all options are displayed.
14.	For IPv4 Lease Pool, select the lease pool you created.  
a.	Note:  Just as when creating pool resources in LTM, you can click the + symbol to create a lease pool without having to start over if you had not created the lease pool already.
4.	It will default to “Auto Map” for SNAT Pool, which is the desired setting.
5.	Under Client Settings, Traffic Options, select Use Split Tunneling for Traffic, and enter the subnet 10.20.4.0, with a subnet mask of 255.255.255.0. 
a.	If you left this setting at Force all traffic through tunnel, it would  indeed work, except OOPS!  Once established, the RDP session to your test Win10 VE would disconnect.  If you ever forget and leave Force all traffic through tunnel enabled, and connect.  You can easily recover by going to Access >> Overview on the management GUI, and killing the session, which will terminate the connection, drop the full VPN and restores access to your Win10 VE again.
6.	Do not enter anything in the sections for DNS Address Space, IPv4 Exclude Address Space, or DNS Exclude Address Space.
7.	Make SURE to move the entry named “default-all” in the “Dynamic LAN Address Spaces” so it is in the “Available” column and NOT the “Selected” column.
8.	Check the box by Allow Local Subnet and Allow Local DNS Servers.
9.	Do not check any other boxes except for DTLS.  The DTLS Port will pre-populate to port 4433. 
10.	Click Update at the bottom to save the changes.  
Note:  If you navigate to a different tab at the top without clicking Update, you will lose all changes you made on that page, so always click Update after making changes to a page, even when just navigating to another tab!

11.	Click the tab for DNS/Hosts and enter 10.20.4.2 for the IPv4 Primary Name Server.  If you were to forget this…no name resolution over the VPN!
12.	Click the checkbox for Register this connection’s address in DNS, and click Update.  
13.	There are many other options available, but in this scenario, no other changes are needed from their default values.  A basic Network Access resource has been created.
6.	Create a new Access Profile of type All (or copy the one you created in Portal Access, and name it Mod6_Network_Access.
1.	Use an Advanced Resource Assign to assign the Network Access resource you created, and a Full Webtop.  Your completed access policy should look as below:  

 
7.	Save your changes and close the VPE
8.	Create a new virtual server for the APM Policy named vs_Net_Access, using the IP address for vpn.apmjs2.local in the appendix, using port 443, and ensure you select the tcp, http, Wildcard ClientSSL profile you created, serverSSL, SNAT Automap, standard rewrite profile, and standard connectivity profile.  Also select the Access Profile you created above, Mod6_Network_Access and save the new virtual server
9.	You are going to create three virtual servers total, so you have two more, but they use the same IP, and need minimal configuration. One to attach your APM Access Policy which you created in the previous step on 443, one for DTLS (UDP!) on port 4433, and one port 80 for a simple redirect to https.  All three virtual servers will use the IP address found in the Appendix for vpn.apmjs2.local.

You created the first virtual server, vs_Net_Access, with its required profiles.
1.	For the DTLS virtual server:
a.	Name: vs_Net_Access_DTLS
b.	Configure it to use the same IP address as vs_Net_Access and port 4433.
c.	Ensure you select the UDP protocol, UDP Protocol Profile, select the SAME Wildcard ClientSSL profile you set for vs_Net_Access, serverSSL, and the same connectivity profile you selected for the port 443 virtual server for the APM Policy.  This is how it is linked to the vs_Net_Access virtual server to use DTLS.
d.	Select SNAT Automap, and save the virtual server.
e.	You do not select an access profile in a DTLS virtual server, as users cannot login via this VS.  
Note:  Do NOT EVER select a VDI Profile or check the box for Application Tunnels (Java&PerApp VPN) on the DTLS VS, as this is incompatible with this VS and will cause your Network Access connection to fail on every connection attempt with little logged info.  This option CAN be selected on your 443 virtual server, but NOT in the DTLS virtual server.  

2.	For the last, a Standard VS, the port 80 redirect virtual server:
a.	Name: vs_Net_Access_Redirect
b.	Configure it to use the same IP as the virtuals above and port 80.
c.	It should be set for the TCP protocol and have the default HTTP profile set.  Save the new virtual server
d.	After you create the vs, edit the port 80 virtual server you created and click on the Resources tab.  
e.	Click “Manage” in the iRules section, and move the system iRule _sys_https_redirect to the Enabled column, and click Finished at the bottom.  
f.	This will redirect any connection to the IP for vpn.apmjs2.local on port 80 to port 443 automatically.
10.	This completes a standard Network Access configuration.  

Note:  If you plan on adding portals to be on your webtop along with your Network Access, ensure you added the default Rewrite profile on your 443 virtual server.  If you do not, APM will not let you assign a portal resource in the Access Profile VPE, so for these labs, setting this is recommended.  Assigning a Rewrite profile will not harm a policy which doesn’t require it. Also be sure to include the SSO Credential Mapping object after the AD Auth or SSO will not work for Portals.

11.	Test Network Access using your browser and logging in with any username or the administrator account by opening your browser, and only typing in vpn.apmjs2.local.  If you didn’t create the redirect VS, then you will need to go to h  
1.	If your clientSSL profile has the wildcard cert and key set correctly in your 443 and 4433 virtual servers on the same IP, you will be redirected to https, and will reach a logon prompt without any certificate warning.
12.	Login with the user:  administrator and pass: abc123!@    
1.	You should then see your full webtop, click your Network Access resource to launch and connect the VPN.  
2.	If you received any certificate warnings logging into the webtop, you will receive several warnings when you launch Network Access and have to click ok to in order to continue.  This is why it is recommended to correct any clientSSL profile issues before testing.   The VPN should connect.

Note: Once connected with Network Access, if prompted for Network Location in Windows, always select “Domain/Work Network”, a private network.  Essentially anything except a “Public” network.

13.	Are www1, AD1, and all other servers reachable using their hostnames?  Example:
C:\ping ad1   Or    C:\ping ad1.apmjs2.local         yes  /  no  
(They should, unless you missed entering the DNS IP 10.20.4.2 in the DNS/Hosts tab of Network Access).  
1.	If you cannot ping by name, verify you can ping 10.20.4.2 by IP. 
2.	If you can ping by IP’s but not by name, you can be assured the problem is a DNS configuration issue in your setup.  
3.	If you cannot ping by IP, ensure you have Automap set in the SNAT setting in Network Access.  Even if you had set SNAT Automap set on your 443 and 4433 virtual servers.  It would not affect Network Access!  If you make a change, you must disconnect and reconnect to retest.
14.	Disconnect the VPN and close your browser.  Assuming you were able to ping machines behind the APM by name.  You did it!  Congratulations!
15.	Review the log messages associated to your session in a session report, and familiarize yourself with the messages.  Many should be quite verbose about what they are doing.
Network Access SNAT’ing
16.	You can easily test Network Access SNAT behavior.  On APM, just use tcpdump to the CLI window in Putty to determine the source address of the Windows client for each of the following steps.  However…if you remember how SNAT behaves in LTM, it operates in EXACTLY the same way here.   Except SNAT for a VPN is set in Network Access, not the virtual server. To change the SNAT setting, navigate to your Network Access resource, and you enable/disable SNAT options there.  The SNAT setting on the virtual server has NO EFFECT on Network Access. 

However, customers will almost always have both Portals and other Resources on a Full Webtop (which require SNAT Automap or SNAT pool enabled on the VS to function), as well as Network Access on the same webtop.  Therefore, in most cases you should always have SNAT Automap or SNAT Pool set on an APM virtual server, and the customer can decide how, or if, they wish to set SNAT on their Network Access. 

Pro Tips!  Common Network Access issues and their resolution!

Should a customer open a case claiming their Network Access works with SNAT Automap on, but fails when it is set to “none”, and they need SNAT disabled.  They have simply not included a route on their network to return the traffic to the BIG-IP.  Solution?   

Ask them to run a traceroute command to ANY IP within the Network Access Lease Pool FROM any internal machine which they could ping fine using Network Access when SNAT Automap was enabled for Network Access.  The trace will almost certainly go to its default gateway which demonstrates asymmetric routing!  The trace “should” go to the internal self IP of the BIG-IP (floating internal if an HA pair), and die right there.  So if it does not, just advise they need to add a route in their routing infrastructure so all traffic for the range of lease pool IP’s are routed to the BIG-IP’s internal self IP.  Once this is done, when SNAT is set to none, traffic from VPN clients will return to where they originated…the BIG-IP.

They also “can” assign lease pool IP’s within the range declared by their internal self-IP’s subnet mask.  However, they of course cannot be in use by any other machine at any time!  If this is done, they can set SNAT to “none”, but they must check the box by “Proxy Arp” in the Network Settings tab in the Network Access resource.  If this is used, as long as machines on their network could ping the internal self IP of the BIG-IP, they will function just as if SNAT Automap were enabled, but each client will retain their assigned lease pool IP’s for unique identification of clients.  The BIG-IP will ARP for all IP’s in the lease pool range.  Which is why they must be inside the subnet of the internal VLAN self IP which is assigned.  If you have any questions on resolving routing issues, or the behavior of Proxy ARP, please do ask your instructor!  This, and putting invalid data in the DNS address space are very common issues, and easy to identify and solve.  So you want to be able solve them easily should it come up in a case!

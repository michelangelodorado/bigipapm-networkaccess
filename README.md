# bigipapm-networkaccess

Manual: BIG-IP Access Policy Manager: Network Access
https://techdocs.f5.com/en-us/bigip-16-1-0/big-ip-access-policy-manager-network-access.html

The BIG-IP Access Policy Manager network access feature provides secure access to corporate applications and data using a standard web browser, or the BIG-IP Edge Client. Using network access, employees, partners, and customers can have access to corporate resources securely, from any location. 

The network access feature provides users with the functionality of a traditional IPsec VPN client. Unlike IPsec, however, network access does not require any pre-installed software or configuration on the remote user's computer. It is also more robust than IPsec VPN against router and firewall incompatibilities.

<img width="468" alt="image" src="https://github.com/michelangelodorado/bigipapm-networkaccess/assets/102953584/f88cafd6-4d29-4d58-8b19-056dcb30d16e">

Use this above dependency diagram and steps below to create and review the objects required to manually configure a working Network Access configuration.  Many objects like Connectivity Profiles can be reused. 

Objects in RED on the dependency diagram are only required when Data Transport Layer Security (DTLS) is enabled or the HTTP redirect option is desired.  The HTTP redirect is just a simple HTTP VS which has a system iRule redirecting users to HTTPS so users do not have to enter HTTPS in their browser to connect and can simply type an FQDN.  Objects in RED italic are not mandatory for Network Access to function, but are recommended.  Please complete them for this lab.

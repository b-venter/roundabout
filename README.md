# Roundabout SIP Proxy
Roundabout is a SIP Proxy that facilitates forwarding of SIP Requests and Responses between endpoints (servers, clients) that do not  support each others’ network transport protocols. For example, a soft client might only support WSS (Secure WebSockets). And the SIP server only supports TCP and UDP. Using Roundabout as a proxy allows the client to still communicate with the SIP Server.

As with a traffic roundabout, Roundabout SIP Proxy has multiple entries and exits – but in general leaves the traffic content unchanged.

The rules for routing are logically deduced from the destinations in the Request URI, the parameters used to add trusted servers at runtime and the details provided by clients when registering.

### Usage

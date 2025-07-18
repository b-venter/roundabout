# Roundabout SIP Proxy
<img width="356" height="347" alt="image" src="https://github.com/user-attachments/assets/5519bb5a-4816-41d3-81a5-02e2e0ea7894" />


Roundabout is a SIP Proxy that facilitates forwarding of SIP Requests and Responses between endpoints (servers, clients) that do not  support each others’ network transport protocols. For example, a soft client might only support WSS (Secure WebSockets). And the SIP server only supports TCP and UDP. Using Roundabout as a proxy allows the client to still communicate with the SIP Server.

As with a traffic roundabout, Roundabout SIP Proxy has multiple entries and exits – but in general leaves the traffic content unchanged.

The rules for routing are logically deduced from the destinations in the Request URI, the parameters used to add trusted servers at runtime and the details provided by clients when registering.

## Usage
There are two main use cases for Roundabout:  
- **Server to server:** Typically SIP trunks. Roundabout is specified as a proxy on both servers' configuration. Thus Roundabout receives the SIP Request from one server, sees the R-URI is addressed to the other server, and sends it out via that servers required protocol. Contact Header is edited to reflect the change in transport protocol.
- **Client to server:** In Roundabout terms, a client is any endpoint that is dynamically added/made known to a server by the REGISTER method. This is usually a telephone or soft client - but could also be a PBX or server whose contact details are only determined by what it registers with the Service Provider. It implies that no static configs exist - and thus the server has no means of specifying a proxy to use for requests destined for that client. Normally requests are sent to the Host specified in the Contact Header at time of registering. Roundabout inserts itself in place of the client when registering, so that the Server will send new requests to it first. Roundabout then determines which client the request is meant for by examining the User in the R-URI, which it looks up in its 'client cache' to determine the client's Host value. The client cache is populated dynamically and is not permanent (currently). Thus, if Roundabout is restarted the client cache is emptied: restart the client endpoints to get them to re-register and thereby populate Roundabout's client cache. If a client is not found in the client cache, it will fail to send and receive SIP requests.

The good news is that both use cases can be active simultaneously.

### Server to server
You will need to inform Roundabout of the two or more servers. Here would be a typical command:  
``
### Client to server

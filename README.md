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

- **Certificate Signing Request:** The binary can be run once-off to create a Certificate Signing Request. The TLS key must be provided. If one does not exist, use openssl to create one. RSA and ECDSA keys are both supported.

### Server to server
You will need to inform Roundabout of the two or more servers. Here would be a typical command:  
`./roundabout -local-ip 203.0.113.5 -local-fqdn proxya.example.com -strict-host -strict-client -ack-ruri  -trusted-server 203.0.113.20:server.example.com:5060:tcp:,203.0.113.110:pbx.example.com/srv.tel.example.com:5060:wss:`  

`-local-ip 203.0.113.5`: we need to inform Roundabout as to the IP addrss that belongs to it. No auto sensing of IP addresses is done. This is used not only for specifying the transport layer to transmit the SIP packet, but also as part of the logic to determine whether R-URI might reference Roundabout itself.  
`-local-fqdn proxya.example.com`: same as with specifying local IP, just with reference to Roundabout's FQDN/SIP domain.  
`-strict-host`: This will cause the routing logic to verify whether the Host value in the Request URI matches a server we specified. If omitted, Roundabout will attempt to deliver the request direct to the host in the Request URI even if it does not know it.  
`-strict-client`: it seems strange to include this for 'server to server'. However, Roundabout will always be listening for REGISTER requests to add to the client cache. By specifying it to be strict, it will also see whether the REGISTER request is from a trusted IP address - which we did not specify. So all REGISTER requests will be seen as invalid and not populate the client cache.  
`-ack-ruri`: This toggles whether Roundabout should proxy ACK requests to Hosts specified in the To header or Request URI (RURI). If this flag is omitted, the To header will be used. Recommended to use RURI.  

`-trusted-server`: This flag does more than work with `-strict-host`. This is fundamental to Roundabout routing requests using the correct network protocol. Let's spend a bit of time on the structure of this flag.  
- First, note that multiple server datasets can be provided. Each dataset MUST be separated by a comma (no spaces).
- Each dataset has a strict structure, with ':' separating each set. (ip-address:fqdn-list:port:network-protocol:route-list). Even if omitting a set of data, the corresponding ':' must be present.
- Some set items can have multiple values (fqdn, route). In such a case, use '/' to separate each item.
- route-list is still in development. In the future this will allow you to add Route headers and program Roundabout to not use the R-URI host as the next hop/target.
- fqdn-list is useful where a server might respond with different FQDN/SIP domains, and you will need to be able to match any of those to verify the server.


### Client to server
`-local-ip 203.0.113.5`: In addition to what is stated in 'Server to server', this value will be set in the Contact Header when Roundabout proxies Requests from the client to the server.  
`-local-fqdn proxya.example.com`: In addition to what is stated in 'Server to server', this value will be set in the Contact Header if `-contact-header-fqdn` is specified.  


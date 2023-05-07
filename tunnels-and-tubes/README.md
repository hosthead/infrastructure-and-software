# Tunnels and Tubes

**DRAFT. This document is pending a test run in the lab to make sure there are no errors.**

Tunnels are useful for connecting two endpoints, and for routing traffic between those endpoints.

*Is this another wireguard tutorial?*

Yes, including how to firewall it properly.

## Case

Alice wants to expose a service running on a single IP in her LAN 192.168.100.0/24 to a service on a single IP in Bob's LAN 192.168.200.0/24. Alice has a static public IP address of 203.0.113.201 and the ability to open ports to an internal address. Bob has a dynamic public IP address.

Alice will act as the "server" so far as Bob will always be initiating the connection, however in wireguard either side can initiate a connection. Since Bob is behind a NAT though, he must initate the connection and send an occasional keepalive to keep the UDP port open.

## Environment

* any Firewall
* any Linux
    * firewalld
    * wireguard

## Alice

Alice's service is Postgres on IP 192.168.100.10 port 5432. She wants to expose this to Bob's service on his LAN at 192.168.200.10 (source port is dynamic).

Alice's subnet 192.168.100.10 is behind a firewall that sits on the LAN and WAN subnets. Alice's firewall is configured to forward UDP port 7444 to LAN IP 192.168.100.10

Alice is running firewalld. 192.168.100.10 is in zone public and port 7444/udp is permitted

### Alice's Wireguard

Alice's wireguard configuration is as follows.

    [Interface]
    PrivateKey= alice's private key
    Address=192.0.2.1/30
    ListenPort=7444
    
    [Peer]
    PublicKey= bob's public key
    PresharedKey= both end's shared key
    AllowedIPs=192.0.2.2/32

The Interface section:

1. PrivateKey= Your private key.  
Generate this with `wg genkey`. Add to the config, then get the public key with `wg pubkey`. Paste the key in, press enter, then press CTRL+D and the public key should be returned. Provide the **public** key from that command to Bob. The public key is not secret, but ensure that Bob receives it without it being tampered with.
2. Address= Sets the address and subnet mask to assign to this end of the tunnel.  
Coordinate with Bob that this is not a subnet that conflicts with his network.
3. ListenPort= The port to bind to (you can't specify the IP to bind to, you must restrict this with firewall rules)  
System services are usually run on ports 1-1024. Dynamic port ranges start as low as 16000 and as high as 32000. A good listen port is somewhere between 1-16000 and not conflicting with something commonly used, thus we pick something in the range of 1025-16000.

The Peer section:

1. PublicKey= other end's public key  
Get this from Bob. This material is not secret, but ensure the public key received from Bob is not tampered with.
2. PresharedKey= both end's shared key  
Generate this with `wg genpsk` and share with Bob through a secure means.
3. AllowedIPs= The IP address and subnet to route to through this tunnel.  
This does not restrict what IPs may come into the tunnel, only what traffic will leave this server through the tunnel. The latter will be addressed with firewalld. Set this to the IP address of Bob's end of the tunnel, 192.0.2.2/32

Alice saves this to /etc/wireguard/wg0.conf and enables it with `sudo systemctl enable --now wg-quick@wg0`.

### Alice's firewalld

Alice's firewalld configuration is built by the following.

    sudo firewall-cmd --permanent --zone=public --add-port=7444/udp
    sudo firewall-cmd --permanent --new-zone=bob
    sudo firewall-cmd --permanent --zone=bob --add-interface=wg0
    sudo firewall-cmd --permanent --zone=bob --add-rich-rule='rule family="ipv4" source address="192.0.2.2/32" destination address="192.0.2.1/32" port port="5432" protocol="tcp" log level="info" accept'
    sudo firewall-cmd --reload

This will permit bob to only access postgres on 192.0.2.1:5432 and nothing else.

## Bob

Bob's service wants to connect to Alice's postgres server.

### Bob's wireguard

Bob's wireguard configuration is as follows.

    [Interface]
    PrivateKey= bob's private key
    Address=192.0.2.2/30
    
    [Peer]
    PublicKey= alice's public key
    PresharedKey= both end's shared key
    AllowedIPs=192.0.2.1/32
    Endpoint=203.0.113.201:7444
    PersistentKeepalive=5

The Interface section:

1. PrivateKey= Your private key.  
Generate this with `wg genkey`. Add to the config, then get the public key with `wg pubkey`. Paste the key in, press enter, then press CTRL+D and the public key should be returned. Provide the **public** key from that command to Alice. The public key is not secret, but ensure that Alice receives it without it being tampered with.
2. Address= Sets the address and subnet mask to assign to this end of the tunnel.  
Coordinate with Alice that this is not a subnet that conflicts with her network.

There is no ListenPort= because Bob will always initiate the tunnel.

The Peer section:

1. PublicKey= other end's public key  
Get this from Alice. This material is not secret, but ensure the public key received from Alice is not tampered with.
2. PresharedKey= both end's shared key  
Get this from Alice over a secure means.
3. AllowedIPs= The IP address and subnet to route to through this tunnel.  
This does not restrict what IPs may come into the tunnel, only what traffic will leave this server through the tunnel. The latter will be addressed with firewalld. Set this to the IP address of Alice's end of the tunnel, 192.0.2.1/32
4. Endpoint= The endpoint to connect out to, in this case Alice's end.
5. PersistentKeepalive= Send a packet every x seconds if there is no other traffic.  
This is necessary to keep the tunnel alive and keep Alice's endpoint updated if Bob's dynamic address changes.

Bob saves this to /etc/wireguard/wg0.conf and enables it with `sudo systemctl enable --now wg-quick@wg0`.

### Bob's firewalld

Bob's firewalld configuration is built by the following.

    sudo firewall-cmd --permanent --new-zone=alice
    sudo firewall-cmd --permanent --zone=alice --add-interface=wg0
    sudo firewall-cmd --reload

This creates a new zone to hold Alice's interface. No traffic will be permitted inbound from Alice over the tunnel unless it's a response to an outbound request.

## Alice and Bob

Alice and Bob both check that things are working with "sudo wg show wg0 | grep handshake". They see that they have handshaked successfully.

    alice@postgres:~$ sudo wg show wg0 | grep handshake
      latest handshake: 1 minute, 38 seconds ago
    
    bob@somewhere:~$ sudo wg show wg0 | grep handshake
      latest handshake: 1 minute, 39 seconds ago

Bob can at this point talk to Alice's postgres server successfully.

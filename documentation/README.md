# Documentation

A little self referential, innit?

## Some useful IETF RFC's

[RFC5737](https://datatracker.ietf.org/doc/html/rfc5737) IPv4 Address Blocks Reserved for Documentation

Describes IP addresses that can be used in documentation. Useful for avoiding RFC1918 addresses when trying to describe non-private address space in documentation.

    192.0.2.0/24 (TEST-NET-1)
    198.51.100.0/24 (TEST-NET-2)
    203.0.113.0/24 (TEST-NET-3)

[RFC1918](https://datatracker.ietf.org/doc/html/rfc1918) Address Allocation for Private Internets

Describes private IP address space.

       10.0.0.0/8
     172.16.0.0/12
    192.168.0.0/16

## Some useful kernel.org docs

[bonding](https://www.kernel.org/doc/Documentation/networking/bonding.txt) Linux Ethernet Bonding Driver HOWTO

Describes the Linux end of bonding two or more ethernet adapters for redundancy/load balancing.

# Dell OS10 Notes / Gotchas
Notes and gotchas for Dell OS10 (mostly for S4112F-ON switches)

# Gotchas (as of OS10 10.5.1.4)

## IPv6

* When using IPv6 with subnet masks between /64 and /128, be sure to set `hardware l3 ipv6-extended-prefix` with either 1024, 2048 or 3072 otherwise these addresses will not be routable. This setting requires a reboot to come into effect.

## Serial Console

* The OS10 documentation for the S4112F-ON switch states support for 115200 baud rate, 8 data bits, and no parity. However, this functionality is achieved by a Debian process (agetty) with the following command `/sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220` and is spawned by jobs defined in `/etc/init/ttyS0.conf` and `/etc/init/ttyS1.conf`. As far as I can tell, ttyS0 is the standard serial console and ttyS1 is the USB serial console. If connected to a console server with a 9600 baud rate limitation, it will work if you send multiple BREAKS on the console (initially displaying garbage) which cycles through 115200,38400,9600.

## BGP

* When defining as-path filters with `ip as-path access-list 1 permit ASN` and you're already familiar with Cisco as-path filters, the functionality differs in that it uses more standardised regex for path matching. For example, use `ip as-path access-list 1 permit .*15169.*` instead of `_15169_`.

* BGP neighbours have `address-family ipv4 unicast` and `address-family ipv6 unicast` immediately on creation. For IPv6 neighbours, you need to explicitly set `no activate` under `address-family ipv4 unicast` unles you want IPv4 prefixes to also be sent over the IPv6 session.

* If you are coming from a Cisco environment where you are using `ttl-security hops 1` you will instead need to use `ebgp-multihop 255` on the OS10 platform when defining a neighbour in order to achieve ttl security.

* Attaching communities to exported prefixes differs slightly from Cisco and can be achieved by defining an `ip community-list` and using this within your `route-map`. For example: `ip community-list standard LIST1 permit ASN:TAG`. Then use `set comm-list LIST1 add` in your `route-map`.

## IP Access Lists

* If you are used to defining an `ip access-list` in a Cisco environment. You may not have been adding a deny after your permit lines due to there being an implicit deny. However, on the Dell platform there is no implicit deny so you will need to add a deny at the end of your list, for example: `ip access-list ssh_inbound seq 100 deny tcp any any eq 22`.
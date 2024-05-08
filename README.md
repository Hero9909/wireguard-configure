[![Build Status](https://travis-ci.org/endeav0r/wireguard-configuration.svg?branch=master)](https://travis-ci.org/endeav0r/wireguard-configuration)

# wireguard-configure

`wireguard-configure` is a command-line utility to help manage wireguard configurations. It assumes a basic setup with one node acting as a, "Router," and several clients which connect and route traffic between the central router node. It allows you to generate and dump wireguard configurations, and bash scripts which also configure interfaces and routes.

You must have the commandline tool `wg` accessible through your path. This is used to automatically generate private/public wireguard keys.

Configurations are stored in yaml, and can be modified from the command line, or directly in the yaml file.

```
$ wireguard-configure --help
wireguard-configure 0.0.1
Alex Eubanks <endeavor@rainbowsandpwnies.com>
Simple wireguard configuration

USAGE:
    wireguard-configure [FLAGS] <CONFIG> [SUBCOMMAND]

FLAGS:
        --example    Generate an example configuration file
    -h, --help       Prints help information
    -l, --list       List clients in this configuration
    -V, --version    Prints version information

ARGS:
    <CONFIG>    wireguard-configure configuration file

SUBCOMMANDS:
    add-client       Add a client to the configuration
    client-config    Dump client config
    help             Prints this message or the help of the given subcommand(s)
    remove-client    Remove a client from the configuration
    router-config    Dump router config
```

# Example usage:

Generate an example configuration file, run `wireguard-configure --example <filename>`

```
$ target/debug/wireguard-configure --example test.conf
Configuration saved to file
$ cat test.conf
---
master_subnet: ~
router:
  name: "vpn-router"
  private_key: "iPuEMY6qKGkMbiIr9mnXGe6vttctAJkZc0uyrpJqHkk="
  public_key: "qPryNlEW7Le9/S2WsfoaiQugZom6ObW/R1SoxyysO3w="
  external_address:
    address: vpn.com
    port: 47654
  internal_address: 10.0.0.1
clients:
  - name: "client-a"
    private_key: "kFBRdLxeEzjmOVoBj1obUYP/4GQi3Zl1yaw+KDmEKlA="
    public_key: "Sa4P5q5cxZr/oXzR3FoeoOEBSxzNl9+6XSvyd/t48HQ="
    preshared_key: "yNy4vauiqapjofmvp1KRjNH0aVqWmni8yIj1Ek2cmkc="
    external_address: ~
    internal_address: 10.0.1.1
    allowed_ips:
      - 10.0.1.0/24
    persistent_keepalive: 25
  - name: "client-b"
    private_key: "SEEJJHOkw50c7qBO5Rlt+9jybSSosLPEBzTRO/+Gq3w="
    public_key: "oUHLjDjJT3oSc+kKorb9jXWjgjw+dk4V/fg4/SJoIFc="
    preshared_key: "iNHFOA3tOUPU4apAIrAKO+twYsVhyC7xRbSOGFh7Zl4="
    external_address: ~
    internal_address: 10.0.2.1
    allowed_ips:
      - 10.0.2.0/24
    persistent_keepalive: 25
```

We can add another client with the `add-client` subcommand.

```
$ wireguard-configure test.conf add-client --help
wireguard-configure-add-client 
Add a client to the configuration

USAGE:
    wireguard-configure <CONFIG> add-client [OPTIONS] --internal-address <INTERNAL_ADDRESS> --name <NAME>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -a, --allowed-ips <ALLOWED_IPS>                      An comma-delimited list of subnets for this client
    -i, --internal-address <INTERNAL_ADDRESS>            Internal address for the new client
    -n, --name <NAME>                                    Name for the new client
    -p, --persistent-keepalive <PERSISTENT_KEEPALIVE>    Optional persistent keepalive for the client
    -r <PRESHARED_KEY>
            can be one of none(default), generate or <KEY> to respectively use no, a generated or given preshared-key
            [default: none]
    -k, --public-key <PUBLIC_KEY>                        Use the given public key, and don't generate keys automatically

$ wireguard-configure test.conf add-client --name test-net -a 10.0.3.0/24 -i 10.0.3.1 -p 25
Client added
$ wireguard-configure test.conf --list
+------------+------------------+-------------+
| Name       | Internal Address | Allowed IPs |
+------------+------------------+-------------+
| vpn-router | 10.0.0.1         |             |
+------------+------------------+-------------+
| client-a   | 10.0.1.1         | 10.0.1.0/24 |
+------------+------------------+-------------+
| client-b   | 10.0.2.1         | 10.0.2.0/24 |
+------------+------------------+-------------+
| test-net   | 10.0.3.1         | 10.0.3.0/24 |
+------------+------------------+-------------+
```

If you just want a single entrypoint into the network, with no subnet, simply leave that option out. This is good for single clients.

```
$ wireguard-configure test.conf add-client --name test-net2 -i 10.0.10.10
Client added
$ wireguard-configure test.conf --list
+------------+------------------+---------------+
| Name       | Internal Address | Allowed IPs   |
+------------+------------------+---------------+
| vpn-router | 10.0.0.1         |               |
+------------+------------------+---------------+
| client-a   | 10.0.1.1         | 10.0.1.0/24   |
+------------+------------------+---------------+
| client-b   | 10.0.2.1         | 10.0.2.0/24   |
+------------+------------------+---------------+
| test-net   | 10.0.3.1         | 10.0.3.0/24   |
+------------+------------------+---------------+
| test-net2  | 10.0.10.10       | 10.0.10.10/32 |
+------------+------------------+---------------+
```

If you want a preshared key to be generated add the option `-r generate`.

```
$ wireguard-configure test.conf add-client --name test-net2 -i 10.0.10.11 -r generate
Client added
$ wireguard-configure test.conf --list
+------------+------------------+---------------+
| Name       | Internal Address | Allowed IPs   |
+------------+------------------+---------------+
| vpn-router | 10.0.0.1         |               |
+------------+------------------+---------------+
| client-a   | 10.0.1.1         | 10.0.1.0/24   |
+------------+------------------+---------------+
| client-b   | 10.0.2.1         | 10.0.2.0/24   |
+------------+------------------+---------------+
| test-net   | 10.0.3.1         | 10.0.3.0/24   |
+------------+------------------+---------------+
| test-net2  | 10.0.10.10       | 10.0.10.10/32 |
+------------+------------------+---------------+
| test-net3  | 10.0.10.11       | 10.0.10.11/32 |
+------------+------------------+---------------+
```

We can now dump ready-to-go configs.

```
$ wireguard-configure test.conf router-config --linux-script
cat > vpn.conf <<EOF
[Interface]
# name: vpn-router
PrivateKey = kHen9MofIs06r3Dw6Bo3VkSelIHMHoVh+DTaXX2LwXE=
ListenPort = 47654
[Peer]
# client-a
PublicKey = BJRQ2ka8JpjkK69S0ZSA2nowJyfQmG3XwzTzC6sqSBg=
PresharedKey = MJRQ8jXaMKVZfRG0sBOqUy/gQbl3EwOiXQ2ucfirAGw=
AllowedIPs = 10.0.1.0/24
[Peer]
# client-b
PublicKey = 1XWU86Ywn4YMb02MLPEPYjtT37TBHhslCJemecUatS8=
PresharedKey = CGAbQP1o8il16cIDKhMi4I2TEpymesUtuhQe4KpIBFo=
AllowedIPs = 10.0.2.0/24
[Peer]
# test-net
PublicKey = lCreJZKyiyJySWoreX1ZgrkZrxbJTXpqqel4rx9OtG4=
AllowedIPs = 10.0.3.0/24
[Peer]
# test-net2
PublicKey = izFqyAOHkKzXuLPsNPw7eD0C2FELnnwB9kQEvIJlhl4=
AllowedIPs = 10.0.10.10/32
[Peer]
# test-net3
PublicKey = OFVWdggsueaENrB8IrkNgWqKqdwT49m7Dsk2V1bzCxU=
PresharedKey = ON61UcA+8UtGIK4eC+C1IHrHgRjHy0KJl7gZ0EjLDFc=
AllowedIPs = 10.0.10.11/32
EOF
ip link add dev wg0 type wireguard
ip address add dev wg0 10.0.0.1/32
wg setconf wg0 vpn.conf
ip link set up dev wg0
ip route add 10.0.1.0/24 dev wg0
ip route add 10.0.2.0/24 dev wg0
ip route add 10.0.3.0/24 dev wg0
ip route add 10.0.10.10/32 dev wg0
ip route add 10.0.10.11/32 dev wg0

```

```
$ wireguard-configure test.conf client-config test-net
[Interface]
# name: test-net
PrivateKey = 6EdJ+47wkQ0Reo4tiehCEtrBFHI8lp71902D/pczb2Y=

[Peer]
# vpn-router
PublicKey = 88UCYOVRyM79kupgHWJf1XCngNnGHBmk3ItrQGtiwxw=
Endpoint = vpn.com:47654
PersistentKeepalive = 25
AllowedIPs = 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24, 10.0.10.10/32, 10.0.10.11/32
$ wireguard-configure test.conf client-config test-net3
[Interface]
# name: test-net3
PrivateKey = YPwASZ+TvrH2cBERZOsbRDx3TE/6IjtyAf/rkO4RXFg=

[Peer]
# vpn-router
PublicKey = 88UCYOVRyM79kupgHWJf1XCngNnGHBmk3ItrQGtiwxw=
PresharedKey = ON61UcA+8UtGIK4eC+C1IHrHgRjHy0KJl7gZ0EjLDFc=
Endpoint = vpn.com:47654
AllowedIPs = 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24, 10.0.10.10/32, 10.0.10.11/32

$ target/debug/wireguard-configure test.conf client-config test-net --linux-script
cat > vpn.conf <<EOF
[Interface]
# name: test-net
PrivateKey = 6EdJ+47wkQ0Reo4tiehCEtrBFHI8lp71902D/pczb2Y=

[Peer]
# vpn-router
PublicKey = 88UCYOVRyM79kupgHWJf1XCngNnGHBmk3ItrQGtiwxw=
Endpoint = vpn.com:47654
PersistentKeepalive = 25
AllowedIPs = 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24, 10.0.10.10/32
EOF
ip link add dev wg0 type wireguard
ip address add dev wg0 10.0.3.1/32
wg setconf wg0 vpn.conf
ip link set up dev wg0
ip route add 10.0.0.1 dev wg0
ip route add 10.0.1.0/24 dev wg0
ip route add 10.0.2.0/24 dev wg0
ip route add 10.0.3.0/24 dev wg0
ip route add 10.0.10.10/32 dev wg0
```
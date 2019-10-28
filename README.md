ansible-wireguard
=========

This role installs wireguard VPN on desired hosts. Tested on Ubuntu 16.04 and 18.04.

Requirements
------------

ifupdown /etc/network/interfaces is used and **must** be installed on target host,
netplan.io is discouraged due to the lack of hook scripts support (pre-up/post-down/etc).
It **must** be disabled and purged.
Native systemd-networkd support will be added in the future as a option.

Role Variables
--------------

`wireguard__tunnels` - the list of wireguard tunnels that will be configured on the host. At least one tunnel must be defined.
By default, one pre-configured tunnel will be set up with default parameters.

#### Tunnel options:

- `name` - the name of the tunnel interface, default tunnel has name `wg0`.
- `address_v4` - ip v4 address of the tunnel. Default tunnel has address *192.168.68.1*
- `netmask_v4` - netmask of v4 address. Default tunnel hap mask *255.255.255.0*. Ignored iv `address_v4` is not set.
- `address_v6` - ip v6 address of the tunnel. Default tunnel has no desired v6 address.  
- `netmask_v6` - ip v6 netmask of the tunnel. Default is 64. It will be ignored if `address_v6` is not set.

You MUST set at least one address (v4 or v6) for your tunnel, else role will fail.

- `private_key` - private wireguard key of your tunnel. Required! Default tunnel has key `0CrLFmHPWBrpmKl42B0QYBhW04EmhObzDqBD7s69L3w=`
you can generate keys with `wg genkey` tool.
- `listen_port` - UDP port of your wireguard tunnel. Required for server/p2p mode, in client mode leave it empty. Default is `8999`

##### Peers

`peers` - this is the list of peers inside your tunnel description. You must have at least one configured peer. Any amount of peers can be confugured for one tunnel.
By default, one peer is pre-confugured.
Peer options:
- `endpoint` - endpoint of the peer. Must be set in format IP:PORT, for example: `12.23.45.67:8999`. Default peer has no endpoint,
so peer can connect from any IP address.
- `keepalive` - PersistentKeepalive option, for example: *25*. For default peer it is not set.
- `public_key` - public key of the peer. Required! Default peer has key `gSfHUONX1PbwxoBJUVInr0ti+TuZgEhGXdLBDOHyGlU=`
- `allowed_ips` - list of allowed IPs. Default list is fully permissive:   
    - 0.0.0.0/0
    - ::/0
- `v4_up_commands`, `v4_down_commands`, `v6_up_commands`, `v6_down_commands` - this is the lists of the Up and Down commands,
will be executed when tunnel interface starts and stops. This commands will be added to the /etc/network/interfaces file.

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts:
        - wireguard
      vars:
        - ansible_python_interpreter: "/usr/bin/python3"
        - wireguard__tunnels:
          - name: wg0
            address_v4: 192.168.68.2
            netmask_v4: 255.255.255.0
            address_v6: fd09:6d14:f8ad:8916::1
            netmask_v6: 64
            private_key: 0CrLFmHPWBrpmKl42B0QYBhW04EmhObzDqBD7s69L3w=
            listen_port: 8999
            peers:
              - endpoint: "1.1.1.1:8006"
                public_key: gJAuwTsauIfXFXgTcelO6Xx+Q8VVHjXkgSn8COu4ZVQ=
                allowed_ips:
                  - 192.168.68.0/24
                  - fd09:6d14:f8ad:8916::/64
          - name: wg1
            address_v4: 192.168.69.2
            netmask_v4: 255.255.255.0
            private_key: 0CrLFmHPWBrpmKl42B0QYBhW04EmhObzDqBD7s69L3w=
            peers:
              - keepalive: 25
                public_key: +ZAWdjZ5zxK4DXPQ2RZ//WQOYKHMggBklRB91jYDRk0=
                allowed_ips:
                  - 192.168.69.0/26
                  - 10.0.3.0/24
              - keepalive: 25
                public_key: y1rOwO2My9BmnaIKXXdqlFbeNCnEo2chgePlroVPKzE=
                allowed_ips:
                  - 192.168.69.64/26
            v4_up_commands:
              - ip route add default dev wg1 table 102
              - ip rule add from 10.0.3.0/28 table 102
            v4_down_commands:
              - ip rule del from 10.0.3.0/28 table 102
              - ip route del default dev wg1 table 102
      roles:
        - wireguard

License
-------

MIT License

Author Information
------------------

Kirill I. Shatalaev, ksh@zombiehost.net

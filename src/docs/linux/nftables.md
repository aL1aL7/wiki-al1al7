# nftables - Firewall

## Basics

Using nftables as the successor to iptables makes life much more convenient. So I switched to nftables at my projects.
For debian (bookworm) it's necessary to enable the nftables service:

```bash
systemctl enable nftables.service
```

Default configuration file for nftables is located at: /etc/nftables.conf. I don't want to make big modifications at preinstalled files, so I just add an include statement for customized rules:

```bash
include "/etc/custom/nftables.rules" 
```

## Basic commands for nftables

```bash title="List loaded ruleset"
nft list ruleset
```

```bash title="List set WHITELIST of table CUSTOM"
nft list set inet CUSTOM WHITELIST
```

## Basic ruleset for a server providing http/https services

```bash
#!/usr/sbin/nft -f

table inet CUSTOM {
    # empty set WHITELIST - will be filled dynamically by scripts
    set WHITELIST {
        type ipv4_addr
    }
    chain INPUT {
        type filter hook input priority 0; policy drop;

        # allow established/related connections
        ct state {established, related} accept

        # early drop of invalid connections
        ct state invalid drop

        # allow from loopback
        iifname lo accept

        # allow icmp (ipv4) - only from IP set WHITELIST
        ip protocol icmp ip saddr @WHITELIST accept

        # allow only ssh connections from IP set WHITELIST
        tcp dport {22} ip saddr @WHITELIST accept

        # allow web services
        tcp dport {80, 443} accept

        # allow internal container interface
        iifname "lxcbr0" ip daddr 10.0.3.1 accept

        # everything else drop
        drop
    }

    chain OUTPUT {
        type filter hook output priority 0; policy accept;
    }

    chain FORWARD {
        type filter hook forward priority 0; policy drop;
        oifname "lxcbr0" accept
        iifname "lxcbr0" accept
    }
}

```

## Updating nftables set

The following bash script updates the set WHITELIST with a dynamic IP address. The script should be executed periodically by a cron job.
The variables `ips` and `host` have to be modified.

```bash
#!/bin/bash
# Update ipset to let my dynamic IP in
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# nftables table and set
ipset=WHITELIST
table=CUSTOM

# dynamical IP for resolving via DNS
host=myurl.example.com

#IP for whitelisting
ips="127.0.0.1 127.0.0.2"

# hostname for logging
me=$(basename "$0")

ip=$(dig +short $host | tail -1)
if [ -z "${ip}" ]; then
    logger -t "${me}" "IP for ${host} not found"
    exit 1
fi

ips="${ips} ${ip}"

ret=`nft get element inet ${table} ${ipset} { ${ip} } > /dev/null 2>&1; echo $?`

if [[ ${ret} == "1" ]] then
    nft flush set inet ${table} ${ipset}
    logger -t "${me}"  "Adding IP ${ip} to set ${ipset}."
    for i in ${ips}
    do
        logger -t "${me}"  "Adding IP ${i} to set ${ipset}."
        nft add element inet ${table} ${ipset} { ${i} }
    done
fi

exit 0

```

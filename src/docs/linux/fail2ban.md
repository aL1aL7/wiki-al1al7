# Fail2Ban

## 1. Unban IP

```bash title="List banned IP via iptables"
iptables -L -n
```

```bash title="Get configured jails"
fail2ban-client status
```

```bash title="Unban IP"
fail2ban-client set JAIL-NAME unbanip XXX.XXX.XXX.XXX
```

## 2. Configuration for caddy reverse proxy

```bash title="Enable access logging at caddy"
(custom_log) {
    log {
        format json {
            time_format iso8601
        }
        output file /var/log/caddy/{args[0]}.access.log {
            roll_size 10mb
            roll_keep 20
            roll_keep_for 720h
        }
    }
}

mydomain.com {
    import custom_log mydomain.com
    reverse_proxy XXX.XXX.XXX.XXX
}
```

```bash title="Create caddy filter file (regex) for fail2ban"
#/etc/fail2ban/filter.d/caddy-status.conf
[Definition]
failregex = ^.*"remote_ip":"<HOST>",.*?"status":(?:401|403|500),.*$
ignoreregex =
datepattern = "ts":"%%Y-%%m-%%dT%%H:%%M:%%S.
```

```bash title="Enable jail in fail2ban config"
#/etc/fail2ban/jail.local
[caddy-status]
backend     = auto
enabled     = true
port        = http,https
filter      = caddy-status
logpath     = /var/log/caddy/*.access.log
maxretry    = 10
```

# Caddy webserver - valid certificates for local services

1. Setup a debian machine or use an existing machine
2. Install xcaddy (add xcaddy apt repository)

    ```bash
    apt install -y debian-keyring debian-archive-keyring apt-transport-https
    curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg
    curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-xcaddy.list
    sudo apt update
    sudo apt install xcaddy
    ```

3. Install prebuilt Go language binaries from the official Go website

    ```bash
    mkdir golang
    cd golang

    # remove existing go version
    rm -rf /usr/local/go

    # check for latest release and download
    wget https://go.dev/dl/go1.22.2.linux-amd64.tar.gz
    tar -xvf go1.22.2.linux-amd64.tar.gz -C /usr/local

    # set environment variables for go / e.g. edit bash.bashrc
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
    ```

4. Build your own caddy binary with needed plugins - in this case include dns-netcup module

    The dns-netcup plugin allows Caddy to automate the process of obtaining SSL certificates using DNS-01 challenges via Netcup's DNS API. This is required when you want to issue certificates for wildcard domains or when your services are not directly accessible from the public internet, as it enables certificate validation through DNS rather than HTTP.

    ```bash
    xcaddy build --with github.com/caddy-dns/netcup
    # move binary to /usr/bin
    # rename it to caddy_custom to mark it as custom build
    mv caddy /usr/bin/caddy_custom
    # create default config Directory for Caddyfile
    mkdir /etc/caddy
    touch /etc/caddy/Caddyfile
    ```

5. Create systemd service file and enable it

    ```bash
    # create caddy group
    groupadd --system caddy

    # create caddy user
    useradd --system \
        --gid caddy \
        --create-home \
        --home-dir /var/lib/caddy \
        --shell /usr/sbin/nologin \
        --comment "Caddy web server" \
        caddy

    # create default working folder
    mkdir /var/lib/caddy
    chown -R caddy:caddy /var/lib/caddy

    # create empty service file
    touch /lib/systemd/system/caddy.service
    ```

    ```bash title="Service file for caddy"
    # caddy.service
    #
    # For using Caddy with a config file.
    #
    # Make sure the ExecStart and ExecReload commands are correct
    # for your installation.
    #
    # See https://caddyserver.com/docs/install for instructions.
    #
    # WARNING: This service does not use the --resume flag, so if you
    # use the API to make changes, they will be overwritten by the
    # Caddyfile next time the service is restarted. If you intend to
    # use Caddy's API to configure it, add the --resume flag to the
    # `caddy run` command or use the caddy-api.service file instead.

    [Unit]
    Description=Caddy
    Documentation=https://caddyserver.com/docs/
    After=network.target network-online.target
    Requires=network-online.target

    [Service]
    Type=notify
    User=caddy
    Group=caddy
    ExecStart=/usr/bin/caddy_custom run --environ --config /etc/caddy/Caddyfile
    ExecReload=/usr/bin/caddy_custom reload --config /etc/caddy/Caddyfile --force
    TimeoutStopSec=5s
    LimitNOFILE=1048576
    LimitNPROC=512
    PrivateTmp=true
    ProtectSystem=full
    AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE

    [Install]
    WantedBy=multi-user.target
    ```

    ```sh title="Enable caddy service"
    systemctl daemon-reload
    systemctl enable caddy.service
    systemctl start caddy.service
    ```

6. Sample Caddyfile  

    ```bash
    # The Caddyfile is an easy way to configure your Caddy web server.
    #
    # Unless the file starts with a global options block, the first
    # uncommented line is always the address of your site.
    #
    # To use your own domain name (with automatic HTTPS), first make
    # sure your domain's A/AAAA DNS records are properly pointed to
    # this machine's public IP, then replace ":80" below with your
    # domain name.
    
    #global options
    {
            # E-Mail for ACME
            email info@mydomain.de
    }
    
    # caddy directive for import
    (insecure) {
            transport http {
                    tls_insecure_skip_verify
            }
    }
    
    # reverse proxy for websockets
    wss01.mydomain.de:8884 {
            @websockets {
                    header Connection *Upgrade*
                    header Upgrade websocket
            }
            tls {
                    dns netcup {
                            customer_number XXXXX
                            api_key XXXXX
                            api_password XXXXX
                    }
                    propagation_timeout 900s
                    propagation_delay 600s
                    resolvers 1.1.1.1
            }
            reverse_proxy @websockets 192.168.2.91:8884 {
                    import insecure
            }
    }
    
    # wildcard domain/certificate for internal services
    *.mydomain.de {
            tls {
                    dns netcup {
                            customer_number {env.NETCUP_CUSTOMER_NUMBER}
                            api_key {env.NETCUP_API_KEY}
                            api_password {env.NETCUP_API_PASSWORD}
                    }
                    # netcup dns is slow
                    propagation_timeout 900s
                    propagation_delay 600s
                    resolvers 1.1.1.1
            }
    
            @ilo01 host ilo01.mydomain.de
            handle @ilo01 {
                    reverse_proxy 192.168.2.13 {
                            import insecure
                    }
            }
    
            @pve01 host pve01.mydomain.de
            handle @pve01 {
                    reverse_proxy 192.168.2.10:8006 {
                            import insecure
                    }
            }
    
            @pbs01 host pbs01.mydomain.de
            handle @pbs01 {
                    reverse_proxy 192.168.2.15:8007 {
                            import insecure
                    }
            }
    
            @home host home.mydomain.de
            handle @home {
                    reverse_proxy https://192.168.2.94 {
                            import insecure
                    }
            }
    
            handle {
                    abort
            }
    
    }
    ```

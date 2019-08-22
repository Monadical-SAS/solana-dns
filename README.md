# Solana DNS

A simple DNS server that uses the high-performance [Solana](https://solana.com/developers/) blockchain as the backing datastore (with an optional REST API + Web UI).

(Loosely based on https://github.com/hbouvier/dns and https://github.com/solana-labs/example-messagefeed)

## Quickstart

```bash
git clone https://github.com/pirate/solana-dns
cd solana-dns
npm install
npm run server --bind-dns=127.0.0.1:5300 --bind-http=127.0.0.1:5380 --upstream=1.1.1.1,8.8.8.8
```

**To query the DNS server:**
```bash
dig @127.0.0.1 -p 5300 google.com
```

**To view the Web UI:**  
Open http://127.0.0.1:5380

**To query the REST API:**
```bash
curl http://127.0.0.1:5380/dns/api/v1/name/google.com
```

## Configuration

#### `--bind-dns`

**Format:** `--bind-dns=[host]:[port]`  
**Example:** `--bind-dns=0.0.0.0:53`  
  
**Default:** `--bind-dns=127.0.0.1:5300`  

**To run the server on port 53 (the standard DNS port):**

1. Check to see if a DNS server is already running on `127.0.0.1:53`
    ```bash
    sudo nc -ulp 53 || echo "port already in use"
    ```
2. On Ubuntu: you may have to stop `systemd-resolvd` (the system DNS resolver that binds to `:53` by default)
    ```bash
    # to stop it immediately, run:
    systemctl stop systemd-resolvd
    
    # then, if you want to prevent it from starting the next time you reboot, run:
    systemctl disable systemd-resolvd

    # make sure /ets/resolv.conf exists and has at least one non-local upstream DNS server
    echo "nameserver 1.1.1.1" >> /etc/resolv.conf
    ```
2. Once the port is confirmed open, you can bind Solana-DNS to `:53`
    ```bash
    # listening on ports below 1000 requires sudo on most systems

    # to run a server that responds to DNS queries from localhost only, run:
    sudo npm run server --bind-dns=127.0.0.1:53

    # to run a server that responds to DNS queries from your local network only, run:
    sudo npm run server --bind-dns=x.x.x.x:53  # replace x.x.x.x with your LAN IP

    # to run a public server that responds to DNS queries from any location, run:
    sudo npm run server --bind-dns=0.0.0.0:53
    ```
3. Optional: tell your system to use the local DNS server for all queries:
    
    ```bash
    # edit /etc/resolv.conf and put this line *above* any other "nameserver x.x.x.x" lines
    nameserver 127.0.0.1
    ```
    On macOS you can set this under `System Preferences > Networking > Advanced > DNS > +`.

#### `--bind-http`

**Format:** `--bind-http=[host]:[port]`  
**Example:** `--bind-http=127.0.0.1:5380`  
  
**Default:** `--bind-http=off`  

Specify the local [ip]:[port] to bind the web UI server to. It must be `0.0.0.0:[port]` in 
order to accept HTTP requests from any client and not just localhost.

#### `--upstream=x.x.x.x,y.y.y.y`

**Format:** `--upstream=[host]:[port],[host2]:[port2],...`  
**Example:** `--bind-dns=208.67.222.222,dns.example.com:5353`  
  
**Default:** `--bind-dns=1.1.1.1,8.8.8.8,208.67.222.222`

Specify which upstream DNS servers to send requests to when the query cannot be resolved via Solana DNS.



## REST API

* GET `/api/dns/A`

    List all host to IP address mappings that the local server knows about.

* GET `/api/dns/A/{domain}`

    Return the IP address for the given given domain.

* PUT `/api/dns/A/{domain}`

    Create or Modify the IP address for the given domain.

* DELETE `/api/dns/A/{domain}`

    Forget the IP address record for the given domain.

* DELETE `/api/dns/A/?force=true`

    Forget all records.

* GET `/api/dns/zone`

    Return the DNS ZONE.

* GET `/api/dns/status`

    Return the DNS server status.

## TODO

- Add more examples for the REST API
- Add support for more DNS record types
- Add support for adding/removing/modifying DNS records via CLI
- Add config file / environment variable support for configuration
- Add config file / environment variable support for hardcoded DNS records

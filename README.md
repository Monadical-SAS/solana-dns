<div align="center">
<img src="https://i.imgur.com/mOGmEPW.png" alt="Logo" width="400px"/>
<h1>Solana DNS</h1>
<br/>
<b>A simple DNS server using the high-performance <a href="https://solana.com/developers/">Solana blockchain</a> to store records.</b>
<br/>
(Implemented in <2000 lines of JavaScript, w/ an optional REST API + Web UI)
<br/><br/>
<sup>(Based on code from <a href="https://github.com/hbouvier/dns">hbouvier/dns</a> and <a href="https://github.com/solana-labs/example-messagefeed">solana-labs/example-messagefeed</a>)</sup>
<br/><br/>
<img src="https://img.shields.io/badge/License-MIT-blue">
<img src="https://img.shields.io/badge/Status-Proof_of_Concept-red">
<img src="https://img.shields.io/badge/Powered_by-Solana-02FEAD">
</div>

---

## Quickstart

**Install the language dependencies:**

 - Rust https://www.rust-lang.org/tools/install
   `curl https://sh.rustup.rs -sSf | sh` / `apt install rustup` / `brew install rustup`
 - JavasScript https://nodejs.org/en/download/package-manager/
    `apt install node` / `brew install node`
 - Docker https://docs.docker.com/install/
    Required because parts of the Solana build process run in Docker.

**Clone the repo & install the project dependencies:**
```bash
git clone https://github.com/pirate/solana-dns
cd solana-dns
npm install
```

**Run the server on localhost:**
```bash
npm run server --bind-dns=127.0.0.1:5300 --bind-http=127.0.0.1:5380 --upstream=1.1.1.1,8.8.8.8
```

**To query the DNS server:**
```bash
dig @127.0.0.1 -p 5300 google.com
```

**To view the Web UI:**
```
Open http://127.0.0.1:5380
```

**To query the REST API:**
```bash
curl http://127.0.0.1:5380/dns/api/v1/name/google.com
```

---

## Configuration

#### `--bind-dns=[host]:[port]`

**Default:** `--bind-dns=127.0.0.1:5300`  
**Example:** `--bind-dns=0.0.0.0:53`  
  
Specify the local [ip]:[port] to bind the DNS server to. It must be `0.0.0.0:53` to act
as a standard public DNS server that can accept requests from any client (instead of just localhost).

See the [instructions below](#start-the-server-on-port-53) if you want to bind to port `53` instead. 

#### `--bind-http=[host]:[port]`

**Default:** `--bind-http=off`  
**Example:** `--bind-http=127.0.0.1:5380`  
  
Specify the local [ip]:[port] to bind the web UI server to. It must be `0.0.0.0:[port]` in 
order to accept HTTP requests from any client (instead of just localhost).

#### `--upstream=[host]:[port],[host2]:[port2],...`

**Default:** `--bind-dns=1.1.1.1,8.8.8.8,208.67.222.222`  
**Example:** `--bind-dns=208.67.222.222,dns.example.com:5353`  

Specify which upstream DNS servers to send requests to when the query cannot be resolved via Solana DNS.

---

## Usage

---

### CLI

#### Start the server

See the [Configuration](#Configuration) section for a list of the options available.
```bash
npm run server [options]
```

To bind to any ports below 1000, most systems require running the program as root.
```bash
sudo npm run server [options]
```

#### Start the server on port 53

By default the DNS server listens on a custom UDP port `5300` in order to avoid requiring `sudo` or conflicting
with any existing local DNS server. To bind to the the standard DNS port (UDP `53`) instead, follow the steps below.

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
    On macOS you can set this under:
    ```
    System Preferences > Network > Advanced > DNS > [+]
    127.0.0.1
    (drag it to the top of the list if there are other entries)
    ```

#### Add/Modify/Delete Records

TODO

---

### Rest API

#### GET `/api/dns/A/{domain}`

Return the IP address for the given given domain.

#### PUT `/api/dns/A/{domain}`

Create or Modify the IP address for the given domain.

#### DELETE `/api/dns/A/{domain}`

Forget the IP address record for the given domain.

#### GET `/api/dns/A/.`

List all host to IP address mappings that the local server knows about.

#### DELETE `/api/dns/A/?force=true`

Forget all records.

#### GET `/api/dns/zone`

Return the DNS ZONE.

#### GET `/api/dns/status`

Return the DNS server status.

---

### DNS over HTTPS API

TODO: implement a Google/Cloudflare-compatible DNS over HTTPS API 

- https://developers.google.com/speed/public-dns/docs/doh/
- https://developers.cloudflare.com/1.1.1.1/dns-over-https/request-structure/

---

### JSON API

TODO: implement a Google/Cloudflare-compatible DNS over HTTPS JSON API

- https://developers.google.com/speed/public-dns/docs/doh/json
- https://developers.cloudflare.com/1.1.1.1/dns-over-https/json-format/

---

## TODO

- Add more examples for the REST API
- Add support for more DNS record types
- Add support for adding/removing/modifying DNS records via CLI
- Add config file / environment variable support for configuration
- Add config file / environment variable support for hardcoded DNS records

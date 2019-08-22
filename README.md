<div align="center">
<img src="https://i.imgur.com/mOGmEPW.png" alt="Logo" width="400px"/>
<h1>Solana DNS</h1>
<br/>
<b>A simple DNS server using the high-performance <a href="https://solana.com/developers/">Solana blockchain</a> to store records.</b>
<br/>
(Implemented in JavaScript and Solana BPF Rust, w/ an optional REST API + Web UI)
<br/><br/>
<sup>(Based on code from <a href="https://github.com/hbouvier/dns">hbouvier/dns</a> and <a href="https://github.com/solana-labs/example-messagefeed">solana-labs/example-messagefeed</a>)</sup>
<br/>
<sub>You can also refer to this project as "who-dat over brick-string", because why not?!<sub>
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

**Setup Solana network access and upload the on-chain side of the program:**

Running code on Solana requires an "account"/wallet with tokens that will be used to run the on-chain part of the DNS server.  
(Similar to how running Ethereum DAPPs on-chain requires spending some tokens in exchange for CPU time)  
Choose which Solana network you want to store your records in:

- Using the public beta testnet (easiest, but all records will be public):
    You automatically get free air-dropped tokens to run code on the beta net. **No real $ needed.**
    ```bash
    npm run signup --net=beta --save-credentials=./secrets.conf
    ```

- Using a localnet (harder, but no data leaves your local machine):
   You get infinite free tokens on your local test net because you own it! **No real $ needed.**
   ```bash
   npm run localnet:update && npm run localnet:up
   npm run signup --net=localnet --save-credentials=./secrets.conf
   ```

- Using the real mainnet (hardest, all records will be public):
   You have to purchase SOL tokens via an exchange to run code on the main net. **Real $ needed.**
   ```bash
   # Not available yet, check https://solana.com/tds/ for updates
   ```

Then upload the Rust BPF program that runs on-chain to handle the storage requests:
```
npm run upload --credentials=./secrets.conf
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

## Architecture

### Data flow

The data flows through the stack like this:

- ⬇️👩‍💻📃 **User**  
    *Makes requests via DNS or HTTP.*  
    `./ui/index.js` (runs in-browser) / direct request via DNS or REST API
 
- ⬇️🖥⬆️ **Local Node Server**  
    *Main JavaScript logic handles CLI commands, DNS queries, and HTTP requests.*  
    `./server/server.js` (runs locally)  / `./server/dns-client.js` (runs locally, resolves any records not found in Solana)
  
- ⬇️🌐⬆️ **Solana Network API**  
    *Calls out via Solana w3 JSON RPC API to the Solana network.*  
    `./network/api.js` (runs locally, calls `https://beta.testnet.solana.com:8443` Solana Network endpoint)  
  
- ⬇️⛓⬆️ **Solana Network Program**  
    *BPF Rust program on Solana network handles reads/writes of stored data.*  
    `./network/kvstore.rs` (gets uploaded and runs on-chain)  
  
- ⬇️➡️⬆️ **Solana Blockchain**  
    *Solana provides the data storage layer to hold the records.*  
    `<Solana internals>` (stored as text blobs on-chain)  

### Execution Flow

The code execution flow looks like this:

0. BPF Rust program gets compiled for for Solana (during setup)   
   `npm run build` -> `./network/build.sh`  
  
2. New Solana account gets created with some free air-dropped tokens (needed to run code on-chain)  
   `npm run signup` -> `./network/signup.js`  
   (only works on testnet/localnet, you're not getting free tokens on mainnet that easy 😉)
  
3. BPF Rust program gets uploaded to the Solana chain under new account  
   `npm run upload` -> `./network/upload.js` -> `./network/kvstore.rs`  
   BPF Loader runs to push program to chain via Solana-provided network endpoint `https://beta.testnet.solana.com:8443` 
  
4. REST API / DNS queries get handled by local node server
   `npm run server` -> `./server/server.js` -> `./server/http-server.js`,`./server/dns-server.js`

5. Local node server calls out to Solana network via Solana Web3 JSON RPC API  
   `./network/api.js` -> `https://beta.testnet.solana.com:8443`
  
6. BPF Rust program runs on Solana network to handle record read/write requests  
   `./network/kvstore.rs` -> `<solana internal API>`
  
7. Solana blockchain handles storage requests  
   `<solana internal API>` -> `<solana blob storage>`

8. If record is found, results are returned back up the stack, if not, they're resolved via the upstream DNS servers
   `./server/dns-client.js` -> `<upstream DNS servers>`

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

**Default:** `--upstream=off`  
**Example:** `--upstream=1.1.1.1,8.8.8.8,208.67.222.222,dns.example.com:5353`  

Specify which upstream DNS servers to send requests to when the query cannot be resolved via Solana DNS.  
The default is `off`, meaning it will return "no result found" if the record is not in the Solana store.  
  
To make it a usable DNS server for all queries, and not just records stored in Solana, it's recommended
to run with a few upstream servers capable of resolving normal internet-level DNS records.

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

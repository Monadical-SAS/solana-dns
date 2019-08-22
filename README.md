<div align="center">
<img src="https://i.imgur.com/mOGmEPW.png" alt="Logo" width="400px"/>
<h1>Solana DNS</h1>
<br/>
<b>A simple DNS server using the high-performance <a href="https://solana.com/developers/">Solana blockchain</a> to store signed records.</b>
<br/>
(Implemented in JavaScript and Solana BPF Rust, w/ an optional REST API + Web UI)
<br/>
<sub>a.k.a "who-dat over very-fast-brick-string"</sub>
<br/>
<br/>
<a href="#Why-Solana-">Why Solana?</a> | 
<a href="#Quickstart">Quickstart</a> | 
<a href="#Documentation">Documentation</a> | 
<a href="#Manual-Setup">Manual Setup</a> | 
<a href="#Architecture">Architecture</a> | 
<a href="#Configuration">Configuration</a> | 
<a href="#Usage">Usage</a>
<br/><br/>
<img src="https://img.shields.io/badge/License-MIT-blue"/>
<img src="https://img.shields.io/badge/Status-Proof_of_Concept-red"/>
<img src="https://img.shields.io/badge/Powered_by-Solana-02FEAD"/>
<br/><br/>
<sup>(Based on code from <a href="https://github.com/hbouvier/dns">hbouvier/dns</a> and <a href="https://github.com/solana-labs/example-messagefeed">solana-labs/example-messagefeed</a>)</sup>
<br/>
<hr/>
<b>WARNING: this project is not actually implemented yet! Follow the <a href="#TODO">TODOs</a> for progress and check back soon for runnable code!</b>
<hr/>
</div>

## Why Solana?

- [Provable history by design.](https://medium.com/solana-labs/proof-of-history-a-clock-for-blockchain-cf47a61a9274)  
  Records are cryptographically provable to have existed at a given time.
  
- [Inherently authenticated.](https://github.com/solana-labs/example-messagefeed#new-user-sign-up)  
  All records are signed with your wallet private key, and can be verified against your public address.  
  (Optional [linking to a Google account](https://github.com/solana-labs/example-messagefeed#new-user-sign-up) can authenticate your account.)
  
- [Inherently immutable.](https://solana-labs.github.io/book/#what-is-a-solana-cluster)  
  Past history can never be modified, whether by accident or maliciously.
  
- [Inherently distributed and fault-tolerant.](https://solana-labs.github.io/book/validator-proposal.html)  
  A globally-distributed network of nodes serve as validators, data replicas, and leaders in the event of a fail-over.
  
- [It's fast as hell.](https://solana.com/solana-milestone-a-multinode-testnet/)  
  The network leader can theoretically handle **710,000 TPS**, and this library additionally caches records locally up to the signed TTL.  

- [It's cool as hell.](https://medium.com/solana-labs/proof-of-history-explained-by-a-water-clock-e682183417b8)  
  Solana ain't your average blockchain, it pulls all this off with **[no sharding or validation delays](https://solana.com/no-sharding-podcast-episode-2-how-does-solana-work/)**.  
  See the [Solana Architecture](#Solana-Architecture) section for more info.

https://diode.io/distributed-infrastructure/Why-DNS-on-Blockchain-is-the-next-step-after-DNS-over-HTTPS-19231/

---

## Quickstart

This installs all the dependencies, creates a free Solana account on the beta testnet, uploads the on-chain code, and runs a local dns server on `127.0.0.1:5300` using Solana as the backing datastore.
```bash
git clone https://github.com/pirate/solana-dns
cd solana-dns

./bin/setup
./bin/server --bind-dns=127.0.0.1:5300 --bind-http=127.0.0.1:5380

dig @127.0.0.1 -p 5300 google.com
```
If you don't like helper scripts installing packages and want to understand/fine-tune the setup, or if you encounter any issues, follow the [Manual Setup](#Manual-Setup) steps below.

---

# Documentation

<a href="#Manual-Setup">Manual Setup</a> | 
<a href="#Architecture">Architecture</a> | 
<a href="#Configuration">Configuration</a> | 
<a href="#Usage">Usage</a>

---

## Manual Setup

**1. Install the language dependencies:**

 - Rust https://www.rust-lang.org/tools/install  
   `curl https://sh.rustup.rs -sSf | sh` / `apt install rustup` / `brew install rustup`
  
 - JavasScript https://nodejs.org/en/download/package-manager/  
    `apt install node` / `brew install node`
  
 - Docker https://docs.docker.com/install/  
    Required because parts of the Solana build process run in Docker.

**2. Clone the repo & install the project dependencies:**
```bash
git clone https://github.com/pirate/solana-dns
cd solana-dns

npm install
```

**3. Create an account on your desired Solana network:**  

Running code on Solana requires an "account"/wallet with tokens that will be used to run the on-chain part of the DNS server.  
(Similar to how running Ethereum DAPPs on-chain requires spending some tokens in exchange for CPU time)  
Choose which Solana network you want to store your records in:

- **Using the public beta testnet** (easiest, all records publicly accessible, **free**):  
    You automatically get free air-dropped tokens to run code on the beta net. 
    ```bash
    ./bin/signup --net=beta --save-config=./secrets.env  # or use --net=edge
    ```

- **Using a local testnet** (harder, no data accessible off your local machine, **free**):  
   You get infinite free tokens on your local test net because you own it!
   ```bash
   ./bin/localnet > localnet.log &
   ./bin/signup --net=localnet --save-config=./secrets.env
   ```

- **Using the public mainnet** (hardest, all records publicly accessible, **real $ needed**):  
   You have to purchase SOL tokens via an exchange to run code on the main net.
   ```bash
   # Not available yet, check https://solana.com/tds/ for updates
   ```

**4. Upload the on-chain side of the program using your account:**  
Build and upload the Rust BPF program that runs on the Solana net to handle requests from your local `solana-dns` server.
```
./bin/build
./bin/upload --config=./secrets.env
```

**5. Run the solana-dns server on localhost:**  
```bash
./bin/server --bind-dns=127.0.0.1:5300 --bind-http=127.0.0.1:5380 --upstream=1.1.1.1,8.8.8.8
```
  
  
**6. You're done! Your `solana-dns` server should be running now. ✅**

 - **To query it via DNS:**
    ```bash
    dig @127.0.0.1 -p 5300 google.com
    ```
  
 - **To query it via the REST API:**
    ```bash
    curl http://127.0.0.1:5380/dns/api/v1/name/google.com
    ```
    
 - **To view the Web UI:**  
    Open http://127.0.0.1:5380

---

## Architecture

<a href="#Data-Flow">Data Flow</a> | 
<a href="#Execution-Flow">Execution Flow</a> | 
<a href="#Solana-Architecture">Solana Architecture</a>

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
    `./network/api.js` (runs locally, calls configured Solana Network's endpoint, e.g. `https://beta.testnet.solana.com:8443`)  
  
- ⬇️⛓⬆️ **Solana BPF Rust Program On-Chain**  
    *The uploaded BPF Rust program spends your account tokens to run on the Solana network and perform reads/writes of stored data.*  
    `./network/kvstore.rs` (gets uploaded and runs on-chain)  
  
- ⬇️➡️⬆️ **Solana Blockchain**  
    *Solana provides the data storage layer to hold the records.*  
    `<Solana internals>` (stored as text blobs on-chain)  

### Execution Flow

The code execution flow looks like this:

1. New Solana account gets created with some free air-dropped tokens (needed to run code on-chain)  
   `./bin/signup` -> `./network/signup.js`  
   (only works on testnet/localnet, you're not getting free tokens on mainnet that easy 😉)
  
2. BPF Rust program gets uploaded to the Solana chain under new account  
   `./bin/upload` -> `./bin/build ./network/kvstore.rs` -> `./network/upload.js ./network/kvstore.rs` 
   BPF Loader runs to push program to chain via configured Solana network's endpoint, e.g. `https://beta.testnet.solana.com:8443` 
  
3. REST API / DNS queries get handled by local node server  
   `./bin/server` -> `./server/server.js` -> `./server/http-server.js`,`./server/dns-server.js`

4. Local node server calls out to Solana network via Solana Web3 JSON RPC API  
   `./network/api.js` -> `https://beta.testnet.solana.com:8443`
  
5. BPF Rust program runs on Solana network to handle record read/write requests  
   `./network/kvstore.rs` -> `<solana internal API>`  
   (Spends account tokens in exchange for the CPU time)
  
6. Solana blockchain handles storage requests  
   `<solana internal API>` -> `<solana blob storage>`

7. If record is found, results are returned back up the stack, if not, they're resolved via the upstream DNS servers
   `./server/dns-client.js` -> `<upstream DNS servers>`

### Solana Architecture

More info on Solana's novel proof-of-history design, architecture, and available APIs can be found here:

#### Intro
- https://solana.com
- https://solana.com/developers/
- https://solana-labs.github.io/book/
- https://github.com/solana-labs/whitepaper/blob/master/solana-whitepaper-en.pdf
- https://medium.com/solana-labs/proof-of-history-explained-by-a-water-clock-e682183417b8
- https://explorer.solana.com/
- https://www.reddit.com/r/solana/

#### Example Code
- https://github.com/solana-labs/example-webwallet
- https://github.com/solana-labs/example-messagefeed
- https://github.com/solana-labs/example-tictactoe

#### Source Code & API Reference
- https://solana-labs.github.io/solana-web3.js/
- https://solana-labs.github.io/solana-web3.js/class/src/bpf-loader.js~BpfLoader.html
- https://github.com/solana-labs/rust-bpf-builder
- https://github.com/solana-labs/solana
- https://github.com/solana-labs/blockexplorer
- https://github.com/solana-labs/solminer
- https://crates.io/users/solana-grimes
- https://github.com/solana-labs/solana/tree/master/client/src
- https://github.com/solana-labs/solana/tree/master/programs
- https://docs.rs/solana-wallet/0.17.2/solana_wallet/wallet/enum.WalletCommand.html
- https://docs.rs/solana-kvstore/0.17.2/solana_kvstore/struct.KvStore.html

---

## Configuration

Config options can be passed to Solana DNS commands (e.g. `./bin/server`) in a few different ways:

1. [Configuration File](#Configuration-File) passed via `--config=path/to/file.env`
2. [Environment Variables](#Environment-Variables) (which override any existing options in the config file)
3. [CLI Parameters](#CLI-Parameters) (which override both env variables and config file params)

### CLI Parameters

#### `--config=path/to/file.conf`

**Default:** `--config=./secrets.env`  
**Example:** `--config=/etc/solana/credentials.conf`  
  
Specify the path to the file containing your network config and account credentials used to connect to a Solana network.

For more info on the config file and options available within, see the [Configuration File](#Configuration-File) section.

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

### Configuration File

When running the server, the path to the config file should be specified via the `--config=path/to/file.env` CLI param.

The config file is initially generated during setup when running:
```bash
./bin/signup --net=beta --save-config=./secrets.env
```
It can also be modified after the initial signup to change the credentials or include some additional options.

The config file must be in Docker/Bash compatible [`.env` format](https://docs.docker.com/compose/env-file/#syntax-rules), and can contain the following parameters:
```bash
SOLANA_NETWORK_NAME=beta
SOLANA_NETWORL_ENDPOINT=https://beta.testnet.solana.com:8443

SOLANA_USER_ID=[unique user id here]
SOLANA_USER_PUBLIC_KEY=[public key here]
SOLANA_USER_PRIVATE_KEY=[private key here]
SOLANA_USER_AUTH_GOOGLE=[optional Google email address here]

# Optionally specify additional DNS server config here
# these options are equivalent to ther respective CLI params
SOLANA_DNS_BIND_DNS=127.0.0.1:5300
SOLANA_DNS_BIND_HTTP=127.0.0.1:5380
SOLANA_DNS_UPSTREAM=1.1.1.1,8.8.8.8,208.67.222.222
```

### Environment Variables

The options in the config file can also be passed as environment varaibles using the same format. e.g.:  
```bash
env SOLANA_DNS_BIND_DNS=127.0.0.1:5300 ./bin/server ...
```

(This works well to pass config when running inside a Docker container)

---

## Usage

<a href="#CLI">CLI</a> | 
<a href="#REST-API">REST API</a> | 
<a href="#DNS-over-HTTPS-API">DNS over HTTPS API</a> | 
<a href="#JSON-API">JSON API</a> | 
<a href="#Web-UI">Web UI</a>

---

### CLI

#### Start the server

See the [Configuration](#Configuration) section for a list of the options available.
```bash
./bin/server [options]
```

To bind to any ports below 1000, most systems require running the program as root.
```bash
sudo ./bin/server [options]
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
    sudo ./bin/server --bind-dns=127.0.0.1:53

    # to run a server that responds to DNS queries from your local network only, run:
    sudo ./bin/server --bind-dns=x.x.x.x:53  # replace x.x.x.x with your LAN IP

    # to run a public server that responds to DNS queries from any location, run:
    sudo ./bin/server --bind-dns=0.0.0.0:53
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

### Web UI

TODO: document Web UI features.

---

## TODO

#### Core Implementation

- [ ] Pull in config loading and CLI env setup boilerplate from https://github.com/pirate/bash-utils
- [ ] Implement `./bin/signup` (based on https://github.com/solana-labs/example-messagefeed/blob/master/src/util/new-system-account-with-airdrop.js)
- [ ] Implement `./bin/upload` to upload the compiled Rust BPF program on-chain (based on https://github.com/solana-labs/example-messagefeed/blob/master/src/server/message-feed.js#L85)
- [ ] Implement `./bin/build` to compile the Rust BPF program (based on https://github.com/solana-labs/example-messagefeed/blob/master/bpf-rust-programs/messagefeed/do.sh)
- [ ] Implement `./network/kvstore.rs` the Rust BPF kvstore + re-usable for other projects (based on https://github.com/solana-labs/example-messagefeed/blob/master/bpf-rust-programs/messagefeed/src/lib.rs)
- [ ] Implement `./network/api.js` to talk to the Rust BPF program running on the Solana network (based on https://github.com/solana-labs/example-messagefeed/blob/master/src/programs/message-feed.js)
- [ ] Implement `./bin/server` -> `./server/server.js` Solana DNS server to call out to API and get/set DNS key:value records (based on https://github.com/hbouvier/dns/blob/master/lib/dns.js)
- [ ] Implement `./server/dns-server.js` to bind to a UDP port and serve DNS queries


#### Lower Priority

- [ ] Add support for more DNS record types besides `A` records (e.g. `CNAME`, `TXT`, etc.)
- [ ] Implement `./server/http-server.js` to bind to a TCP port and serve HTTP Web UI and REST API
- [ ] Implement `./server/dns-client.js` to fetch records from upstream servers when not found on Solana
- [ ] Add support for adding/removing/modifying DNS records via CLI
- [ ] Add config file / environment variable support for hardcoded DNS records
- [ ] Add support for DNS-over-HTTPS API & JSON API
- [ ] Finish documentation for the Web UI and REST, DNS-over-HTTP, and JSON APIs

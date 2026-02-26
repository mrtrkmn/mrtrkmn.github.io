---
title: "Orchestrating a Virtual Hub in Go"
date: 2026-02-26T20:00:00+01:00
tags: ["go", "infrastructure", "docker", "libvirt", "wireguard", "conference", "goroutine", "networking"]
author: "mrturkmen"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Building a single Go binary to spin up containers, VMs, networking, and VPN from one YAML file — why it exists, where it fits, and what's still missing."
disableHLJS: true
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

I recently gave a talk at **Gophers İstanbul 2026** about a tool I built — a single Go binary that spins up Docker containers, libvirt/KVM virtual machines, a full DHCP+DNS stack, and a WireGuard VPN tunnel, all declared in one YAML file. The slides are heavy on code snippets and Go internals. This post is *not* that. Instead, I want to talk about the messy real-world problem behind the tool, where I think something like this is actually useful, what's still broken, and some ideas I haven't had time to explore.

**[📊 Slides from the talk](/talks/orchestrator.html)** · **[💻 Source code](https://github.com/mrtrkmn/orchestrator)**

---

## The Itch

If you've ever needed to demo a multi-tier setup with both containers *and* VMs on a single machine, you know the drill. You write a shell script that calls `docker network create`, then another that generates `dhcpd.conf`, then you manually craft `virsh` XML, then you realize the VM doesn't get an IP because the bridge isn't in promiscuous mode, then you add an `ip link set` call, then someone asks you to reproduce it on a different machine and you discover half of it was hardcoded to your laptop.

I had this exact problem at work. We needed isolated hybrid environments — some services ran in containers, but certain workloads genuinely required a real VM with its own kernel. Setting it up manually took 15-20 minutes and rarely worked the same way twice. Tearing it down was worse. Some containers would linger, the bridge network would survive reboots, and leftover VM definitions would conflict with the next run.

So I built a tool to do it in one shot:

```bash
orchestrator up -c config.yaml    # everything comes up
orchestrator status               # see what's running
orchestrator ssh demo-vm          # SSH in (IP auto-detected)
orchestrator down                 # clean teardown
```

The whole thing compiles into a ~15 MB static binary with no runtime dependencies — you just `scp` it to a server and go.

---

## What It Actually Does

Here's the architecture. One server, one binary, one config file:

<svg viewBox="0 0 760 480" xmlns="http://www.w3.org/2000/svg" style="max-width:720px;margin:1.5em auto;display:block;font-family:'Segoe UI','Helvetica Neue',Arial,sans-serif;">
  <defs>
    <marker id="arrowD" markerWidth="10" markerHeight="7" refX="5" refY="7" orient="auto"><path d="M0,0 L5,7 L10,0" fill="#06b6d4"/></marker>
    <linearGradient id="bg" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#0f172a"/><stop offset="100%" stop-color="#1e293b"/></linearGradient>
    <linearGradient id="bridgeGrad" x1="0" y1="0" x2="1" y2="0"><stop offset="0%" stop-color="rgba(6,182,212,0.08)"/><stop offset="50%" stop-color="rgba(6,182,212,0.22)"/><stop offset="100%" stop-color="rgba(6,182,212,0.08)"/></linearGradient>
    <filter id="glow"><feGaussianBlur stdDeviation="2" result="g"/><feMerge><feMergeNode in="g"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
    <filter id="shadow"><feDropShadow dx="0" dy="2" stdDeviation="3" flood-color="#000" flood-opacity="0.35"/></filter>
  </defs>
  <rect width="760" height="480" rx="16" fill="url(#bg)"/>
  <rect x="16" y="16" width="728" height="350" rx="14" fill="none" stroke="rgba(148,163,184,0.25)" stroke-width="1.5" stroke-dasharray="6 4"/>
  <text x="32" y="38" fill="#94a3b8" font-size="11" font-weight="600" letter-spacing="0.5">SERVER</text>
  <rect x="40" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="109" y="85" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">nginx:alpine</text>
  <text x="109" y="103" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="109" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.10</text>
  <rect x="198" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="267" y="85" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">whoami</text>
  <text x="267" y="103" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="267" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.11</text>
  <rect x="356" y="60" width="138" height="80" rx="10" fill="rgba(6,182,212,0.07)" stroke="#06b6d4" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="425" y="82" text-anchor="middle" fill="#06b6d4" font-size="13" font-weight="700">DHCP / DNS</text>
  <text x="425" y="100" text-anchor="middle" fill="#94a3b8" font-size="10">container</text>
  <text x="425" y="128" text-anchor="middle" fill="#22d3ee" font-size="14" font-weight="700">.5.2 / .5.3</text>
  <rect x="530" y="54" width="194" height="92" rx="12" fill="rgba(244,63,94,0.06)" stroke="#f43f5e" stroke-width="1.5" stroke-dasharray="7 3" filter="url(#shadow)"/>
  <text x="627" y="80" text-anchor="middle" fill="#fb7185" font-size="13" font-weight="700">Debian VM</text>
  <text x="627" y="98" text-anchor="middle" fill="#94a3b8" font-size="10">cloud-init · 1024 MB · 2 vCPU</text>
  <text x="627" y="118" text-anchor="middle" fill="#94a3b8" font-size="10">curl, wget, vim, htop, nmap</text>
  <text x="627" y="136" text-anchor="middle" fill="#fb7185" font-size="10">DHCP assigned IP</text>
  <line x1="109" y1="140" x2="109" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="267" y1="140" x2="267" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="425" y1="140" x2="425" y2="170" stroke="#06b6d4" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="627" y1="146" x2="627" y2="170" stroke="#f43f5e" stroke-width="1.2" stroke-dasharray="4 3" marker-end="url(#arrowD)"/>
  <rect x="30" y="174" width="700" height="40" rx="8" fill="url(#bridgeGrad)" stroke="#06b6d4" stroke-width="1.5"/>
  <text x="380" y="200" text-anchor="middle" fill="#e2e8f0" font-size="14" font-weight="700" filter="url(#glow)">Docker Bridge — 172.19.5.0/24</text>
  <text x="380" y="238" text-anchor="middle" fill="#64748b" font-size="10">gateway .5.1 · mask 255.255.255.0 · DHCP range .5.4 – .5.254</text>
  <line x1="380" y1="214" x2="380" y2="260" stroke="#22d3ee" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <rect x="190" y="264" width="380" height="56" rx="12" fill="rgba(34,211,238,0.06)" stroke="#22d3ee" stroke-width="1.5" filter="url(#shadow)"/>
  <rect x="212" y="282" width="14" height="11" rx="2.5" fill="none" stroke="#22d3ee" stroke-width="1.5"/>
  <path d="M215,282 L215,277 a4,4.5 0 0,1 8,0 L223,282" fill="none" stroke="#22d3ee" stroke-width="1.5"/>
  <text x="240" y="293" fill="#22d3ee" font-size="13" font-weight="700">WireGuard VPN Tunnel</text>
  <text x="240" y="310" fill="#94a3b8" font-size="10">10.10.0.0/24 · X25519 key pair · wg-quick compatible</text>
  <line x1="380" y1="320" x2="380" y2="366" stroke="#22d3ee" stroke-width="1.2" marker-end="url(#arrowD)"/>
  <line x1="380" y1="366" x2="380" y2="410" stroke="#f43f5e" stroke-width="1.2" stroke-dasharray="5 3" marker-end="url(#arrowD)"/>
  <rect x="305" y="416" width="150" height="50" rx="12" fill="rgba(244,63,94,0.08)" stroke="#f43f5e" stroke-width="1.5" filter="url(#shadow)"/>
  <text x="380" y="440" text-anchor="middle" fill="#fb7185" font-size="13" font-weight="700">Remote Client</text>
  <text x="380" y="456" text-anchor="middle" fill="#94a3b8" font-size="10">wg-client.conf</text>
  <rect x="600" y="410" width="12" height="12" rx="3" fill="rgba(6,182,212,0.15)" stroke="#06b6d4" stroke-width="1"/>
  <text x="618" y="420" fill="#94a3b8" font-size="10">Container</text>
  <rect x="600" y="430" width="12" height="12" rx="3" fill="rgba(244,63,94,0.12)" stroke="#f43f5e" stroke-width="1" stroke-dasharray="3 2"/>
  <text x="618" y="440" fill="#94a3b8" font-size="10">VM (libvirt/KVM)</text>
  <rect x="600" y="450" width="12" height="12" rx="3" fill="rgba(34,211,238,0.12)" stroke="#22d3ee" stroke-width="1"/>
  <text x="618" y="460" fill="#94a3b8" font-size="10">VPN Tunnel</text>
</svg>

When you run `orchestrator up`, it creates a Docker bridge, starts DHCP and DNS containers, spins up your application containers and VMs simultaneously (using goroutines — everything runs in parallel), generates WireGuard client configs, and writes a state file so `orchestrator down` can reverse it all cleanly.

The key thing Docker gives you is namespace-isolated containers. The key thing libvirt gives you is a *separate kernel*. Putting both on the same Layer 2 bridge, served by the same DHCP, is the value proposition. A remote client connects over WireGuard and can reach any service — container or VM — transparently.

---

## Where This Is Actually Useful

I haven't built a Kubernetes replacement. This tool is intentionally small and opinionated, and it's genuinely useful for a narrow set of scenarios:

### Training and Workshop Environments

Imagine you're running a 2-day security workshop and every participant needs an isolated network with a web app (container), a vulnerable VM to practice on, and DNS that resolves internal names. Right now you'd either use a cloud provider and spin up per-student VPCs (expensive), or hand everyone a Vagrant setup and pray their laptops can handle it (unreliable). With a beefy shared server, each student gets their own YAML config and a WireGuard tunnel in.

### Edge / IoT Prototyping

When you're prototyping at the edge — maybe a gateway device that needs to run some services in containers but also host a full Linux VM for testing firmware — you don't want to deploy Kubernetes. You want to `scp` a binary, drop a YAML file, and run one command. The cross-compile story makes this trivial: `GOOS=linux GOARCH=arm64 go build -o orchestrator .`

### CI Integration Tests That Need VMs

Some software genuinely can't be tested in a container. Kernel modules, custom networking stacks, anything that needs a real boot sequence. Today, most CI pipelines just skip these tests or use expensive nested virtualization. A tool like this could set up the hybrid environment at the start of a pipeline and tear it down at the end. The idempotent `up`/`down` semantics are designed for exactly this.

### Quick Demo Environments

Sales engineers, solutions architects, support teams — anyone who needs to show a multi-service setup working together on a single machine. Instead of maintaining a fragile demo VM image, you define the environment in a version-controlled YAML file and bring it up in seconds.

### Homelab Orchestration

For the people running Proxmox or bare-metal servers at home — this is a lighter alternative when you have a single node and just want reproducible stacks of containers and VMs without the overhead of a full hyperconverged setup.

---

## How to Leverage It

The config file is where all the thinking happens:

```yaml
network_name: demo-net
subnet: 172.19.5.0/24
network_type: bridge

containers:
  - name: web-demo
    image: nginx:alpine
    ip: 172.19.5.10

  - name: whoami
    image: containous/whoami:latest
    ip: 172.19.5.11

vms:
  - name: demo-vm
    image: ./images/debian-12.qcow2
    memory_mb: 1024
    vcpus: 2
    packages: [curl, wget, vim, htop, net-tools, nmap]

wireguard:
  enabled: true
  peer_name: demo-client
  address: 10.10.0.2/24
```

A few things worth noting about how to get the most out of it:

**Version-control the config, not the environment.** The whole point is that the config file *is* the environment. Check it into Git. When someone asks "what was the demo setup last quarter?", you `git log` rather than trying to remember which containers you ran.

**Use static IPs for services you need to reference.** Containers that other services depend on (your web frontend, your API gateway) should get pinned IPs. The tool supports both static assignment and random allocation from the pool — use randomness for things like worker nodes that nobody addresses directly.

**The VMs get packages baked in.** The `packages` list in the VM config triggers a custom image build — the tool uses cloud-init to install those packages on first boot. This means your VM starts with `curl`, `nmap`, etc. already available, which is a big deal for offline or air-gapped environments.

**WireGuard configs are generated as client-ready files.** After `orchestrator up`, you'll find a `wg-client-<name>.conf` file that you can hand directly to someone. They import it into their WireGuard client and they're in — no manual key exchange.

---

## What's Missing (Honestly)

I presented this at a conference, so naturally I showed the polished bits. Here's what's rough:

### No Graceful Shutdown Chain

Right now, `orchestrator down` stops and removes everything, but there's no ordered shutdown. If your VM needs to flush data before the DHCP container disappears, tough luck. A proper implementation would need dependency ordering — maybe a reverse of the startup DAG. The building blocks are there (context cancellation, WaitGroups), but the sequencing logic isn't.

### Single-Host Only

The tool assumes everything runs on one server. There's no concept of distributing VMs across multiple hosts or federating networks. For many use cases this is fine — it's *supposed* to be a single-host tool — but it limits some of the CI/CD and training scenarios where you'd want to scale out.

### No Health Checks Beyond "Is It Running?"

The `select`-based polling loop checks whether a container has reached "running" state, but that's it. There's no application-level health checking — no HTTP probes, no TCP checks, no readiness gates. If your nginx container is running but misconfigured and returning 502s, `orchestrator status` will happily report everything is fine.

### VM Networking Is Fragile

Getting a VM onto a Docker bridge requires some non-obvious plumbing — putting the bridge in promiscuous mode, using `brctl` to verify the connection, ensuring the bridge helper is configured. The tool handles this, but it's one bad kernel update away from breaking. A more robust approach might use macvlan or a dedicated OVS bridge.

### No Multi-Tenancy

Every run shares the same namespace. If two users run `orchestrator up` with different configs on the same host, they'll conflict on container names, network names, and port bindings. Adding per-user or per-session namespacing (prefixed container names, isolated subnets) would make the training/workshop use case much more practical.

### State Management Is Primitive

The state file is a flat JSON blob. If the binary crashes mid-provisioning, the state file might not reflect reality. There's no reconciliation loop that checks "what's *actually* running?" against "what *should* be running?" — it's fire-and-forget. Compare this to Terraform's state management, and you'll see how much more could be done here.

---

## Interesting Technical Bits

I don't want to rehash every slide (that's what the [presentation](/talks/orchestrator.html) is for), but a few design choices are worth calling out because they surprised me:

**Treating IPv4 addresses as `uint32` changes everything.** Once you represent `172.19.5.10` as a single 32-bit integer, IP pool management becomes arithmetic: range checks are comparisons, broadcast calculation is a bitwise OR, and iterating a subnet is just incrementing. Go's `encoding/binary` and `net` packages make this trivial — no third-party IP manipulation library needed.

**The IP pool's dual strategy is a good example of "know your data."** For a `/24` subnet (254 hosts), you can afford to pre-build a map of all available IPs. For a `/8` (16 million hosts), that's insane — it takes 5 seconds and ~1 GB of memory just to initialize. The pool transparently switches to a lazy strategy for large subnets: only track *used* IPs, generate random candidates, and probe. Result: `/8` initialization went from 5.1 seconds to 0.004 seconds.

**No CGo was a hard constraint.** The main Go libvirt binding (`libvirt-go`) wraps the C library, which means you need `libvirt-dev` installed and lose the static binary. Using `virsh` CLI via `os/exec` is admittedly more fragile (you're parsing text output), but it keeps the binary portable. Same reasoning for WireGuard key generation — `golang.org/x/crypto/curve25519` replaces any need for OpenSSL or `wg genkey`.

**Goroutine-ID logging was a debugging hack that became a feature.** When 6 goroutines are all writing logs simultaneously, it's impossible to tell what's happening without some correlation. Extracting the goroutine ID from `runtime.Stack` output is hacky (you're parsing a debug string), but it works and makes the parallel execution visible in log output. Different IDs + overlapping timestamps = proof that things are actually concurrent, not just concurrent-looking.

---

## Ideas I Haven't Explored Yet

These are things I'd build if I had more time or if someone told me they needed them:

**A `plan` command (like Terraform).** Before `up`, show exactly what will be created — "will create network X, will start containers A, B, C, will define VM D." Let the user review and confirm. Especially useful in production-adjacent scenarios.

**Live reload on config change.** Watch the YAML file and reconcile the running state. Added a new container to the config? Bring it up without tearing down the rest. Removed a VM? Shut it down. This is essentially a control loop, and Go's `fsnotify` + goroutines make it natural to build.

**Plugin system for workload types.** Right now the tool knows about exactly two kinds of workloads: Docker containers and libvirt VMs. But the pattern generalizes — you could imagine plugins for Podman, Firecracker, or even Kata Containers. A `WorkloadDriver` interface with `Start()`, `Stop()`, `Status()` would be a clean abstraction.

**Prometheus metrics endpoint.** Expose current container count, VM count, IP pool utilization, provisioning latency. For a long-running lab server, this would be genuinely useful for capacity planning.

**Snapshot and rollback for VMs.** Define a checkpoint, let students/testers mess things up, then roll back to the known-good state. Libvirt supports snapshots natively — the plumbing is straightforward, it's just not wired up.

**gRPC API.** The CLI is fine for interactive use, but if you want to integrate this into a larger system (a training platform, a CI controller), a gRPC API would let remote clients orchestrate environments programmatically. The Cobra CLI and the `Orchestrator` struct are already cleanly separated, so wrapping `Up()`, `Down()`, and `Status()` in gRPC handlers would be mechanical.

**WireGuard mesh for multi-server.** Instead of a single star topology, allow multiple orchestrator instances to form a mesh. Each generates keys, they exchange public keys and endpoints, and you get a flat network across servers. This is the path toward the distributed version, if it ever makes sense to go there.

---

## Closing Thoughts

This tool exists because Kubernetes is overkill for a single server and shell scripts are undermaintained. The sweet spot is a boring Go binary that does one thing well: bring up a reproducible hybrid environment and tear it down cleanly.

Go turned out to be the right language for this — not because of any single feature, but because infrastructure tooling *wants* the combination of static binaries, cheap concurrency, first-class networking, and a strong standard library. Every major infrastructure project in the ecosystem (Docker, Kubernetes, Terraform, Prometheus) made the same bet.

If any of this sounds useful for your workflow, the code is open and the binary is self-contained. Try it, break it, and tell me what's missing.

**[📊 Talk slides](/talks/orchestrator.html)** · **[💻 GitHub](https://github.com/mrtrkmn/orchestrator)** · **[LinkedIn](https://www.linkedin.com/in/mrturkmen/)**

---
title: "Orchestrating a Virtual Hub in Go"
date: 2026-02-26T20:00:00+00:00
tags: ["go", "infrastructure", "docker", "libvirt", "wireguard", "conference", "goroutine", "networking"]
author: "mrturkmen"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Gophers İstanbul 2026 — Go ile container, VM, ağ ve VPN orkestrasyonu"
disableHLJS: true
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

## Talk: Gophers İstanbul 2026

I gave a talk at **Gophers İstanbul 2026** about building a single Go binary that orchestrates Docker containers, libvirt/KVM virtual machines, networking (DHCP + DNS), and WireGuard VPN — all defined in one YAML file.

**[📊 View the presentation slides](/talks/orchestrator.html)**

**[💻 Source code on GitHub](https://github.com/mrtrkmn/orchestrator)**

---

## What is it?

A ~15 MB Go binary that replaces a pile of shell scripts and manual setup to bring up a complete hybrid environment:

```bash
orchestrator up -c config.yaml    # bring up everything
orchestrator ssh demo-vm          # SSH into a VM
orchestrator status               # check what's running
orchestrator down                 # tear it all down
```

One command creates a Docker bridge network, starts DHCP and DNS containers, provisions application containers, boots a Debian VM with cloud-init, and generates WireGuard VPN config — all in parallel.

---

## Why Go for Infrastructure?

The talk focuses on **why Go's features make it a natural fit for infrastructure tooling**, illustrated with real code from the project. Not "how to write Go" — but which Go features make infrastructure code powerful and natural.

### `net` package — Networking Superpower

Go's standard library provides first-class CIDR parsing and IP manipulation. No third-party libraries needed.

```go
ip, ipnet, err := net.ParseCIDR("172.19.5.0/24")
// → ip=172.19.5.0, ipnet.Mask=ffffff00, err=nil

// Every IPv4 address is just a uint32
base := binary.BigEndian.Uint32(ipnet.IP.To4())
bcast := base | ^binary.BigEndian.Uint32(ipnet.Mask)
// → broadcast = 172.19.5.255
```

The project uses this to build a complete IP pool, calculate broadcast addresses, and generate DHCP configurations — all from a single CIDR string in the YAML config.

### Goroutines — Cheap Parallelism

Starting a goroutine costs ~2 KB of stack memory (vs. ~1 MB for an OS thread). The orchestrator uses a **two-level fan-out pattern**:

- **Level 1:** Two supervisor goroutines — one for all containers, one for all VMs
- **Level 2:** Each supervisor spawns a goroutine per container/VM

This means containers don't wait for VMs, and individual containers don't wait for each other. Total provisioning time = `max(all tasks)` instead of `sum(all tasks)`.

```go
topWG.Add(1)
go func() {                              // Level 1: container supervisor
    defer topWG.Done()
    for _, c := range containers {
        wg.Add(1)
        go func(c ContainerCfg) {         // Level 2: per-container
            defer wg.Done()
            // provision container...
        }(c)  // ← pass loop var as parameter to avoid closure trap
    }
    wg.Wait()
}()
```

### Channels — Safe Communication

Buffered channels collect errors from parallel goroutines without blocking:

```go
errChan := make(chan error, len(containers))  // buffered — won't block senders
// ... goroutines send errors ...
wg.Wait()
close(errChan)
for err := range errChan {
    errs = append(errs, err)
}
return errors.Join(errs...)  // Go 1.20+ — merge multiple errors
```

The project also uses `select` to multiplex context cancellation, timeouts, and polling — three channels in one loop:

```go
select {
case <-ctx.Done():    // cancellation
case <-timer.C:       // timeout
case <-ticker.C:      // poll container status
}
```

### `os/exec` — Infrastructure Compatibility

Go's `exec.Command` with stdin piping lets you drive CLI tools cleanly:

```go
cmd := exec.Command("virsh", "define", "/dev/stdin")
cmd.Stdin = strings.NewReader(xmlConfig)  // pipe XML directly
output, err := cmd.CombinedOutput()
```

And `os.Stat` detects the runtime environment for graceful fallback:

```go
func KVMAvailable() bool {
    _, err := os.Stat("/dev/kvm")
    return err == nil  // no KVM? fall back to QEMU emulation
}
```

### `sync.Mutex` — Thread-Safe IP Pool

The IP pool supports concurrent allocation from multiple goroutines using a dual strategy:

| Subnet | Hosts | Strategy |
|---|---|---|
| /24 | 254 | **Eager** — pre-build map of all IPs |
| /16 | 65K | **Lazy** — random probe, track only used IPs |
| /8 | 16M | **Lazy** — same approach, massive savings |

Result: 10.0.0.0/8 pool initialization went from **5.1s → 0.004s** (1275× speedup).

### Pure-Go Crypto — WireGuard Keys

X25519 key generation using `golang.org/x/crypto/curve25519` — no CGo, no external tools, no OpenSSL dependency. Everything ships in the static binary.

---

## Architecture

```
┌─── Server ────────────────────────────────────────────┐
│                                                        │
│  ┌────────┐ ┌────────┐ ┌──────────┐  ┌─────────────┐  │
│  │ nginx  │ │ whoami │ │ DHCP/DNS │  │  Debian VM  │  │
│  │ .5.10  │ │ .5.11  │ │ .5.2/.3  │  │  cloud-init │  │
│  └───┬────┘ └───┬────┘ └────┬─────┘  └──────┬──────┘  │
│      └──────────┴───────────┴────────────────┘         │
│            Docker Bridge: 172.19.5.0/24                │
│      ┌──────────────────────┐                          │
│      │  WireGuard VPN       │                          │
│      │  10.10.0.0/24        │                          │
│      └──────────┬───────────┘                          │
└─────────────────┼──────────────────────────────────────┘
                  ↓
            Remote Client
```

---

## Key Design Decisions

- **Graceful fallback** — no `/dev/kvm`? Use QEMU. No cloud-init ISO? Skip CD-ROM.
- **IPv4 as uint32** — all IP math is integer comparison, no string parsing in hot paths.
- **Dual-strategy IP pool** — adapt data structure to input size (eager vs. lazy).
- **Error chaining** — `fmt.Errorf("%w")` at every layer; `errors.Join()` for parallel failures.
- **Concurrency testing** — 100 goroutines + `-race` detector on the IP pool.

---

## Links

- **[📊 Presentation slides](/talks/orchestrator.html)**
- **[💻 Source code](https://github.com/mrtrkmn/orchestrator)**
- **[LinkedIn](https://www.linkedin.com/in/mrturkmen/)**

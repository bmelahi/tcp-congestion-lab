# TCP Congestion Lab

**Current version:** 1.0.0 (August 24, 2026)

TCP Congestion Lab is a Mininet-based teaching tool for exploring TCP congestion control, buffering, latency, and competing flows. It creates a dumbbell topology with a shared bottleneck, generates TCP traffic with `iperf3`, samples round-trip time with `ping`, records sender-side TCP state through Linux `ftrace`, captures packets with `tcpdump`, and saves data and plots for analysis.

The teaching package was developed for **COMP 3533: Network Infrastructure and Security** at Mount Royal University. It supports secure, controlled, hands-on experimentation without requiring administrative access to the campus network or a dedicated isolated hardware lab.

> **Environment note:** The original implementation was developed and tested with Mininet 2.3.0 in the Mininet Ubuntu 20.04.1 virtual machine. Within the virtual machine, it requires root access and Linux kernel tracing interfaces. Newer distributions or kernels may require adaptation, especially if the `tcp_probe` tracepoint format differs.

## Original project drafts

Joshua Wolfel's original [tutorial draft](tutorials/original-josh-draft/TLEG_lab.docx) and [VM and Python setup notes](tutorials/original-josh-draft/support/) are preserved as historical working documents. They require further technical and instructional verification before classroom use. Revised tutorial materials are still under development and are not included in this release.

## How the tool works

```text
h1 ----\                         /---- h3
        s1 == shared bottleneck == s2
h2 ----/                         \---- h4
```

For `-n 2`, `h1` sends to `h3` and `h2` sends to `h4`. Each pair has its own flow, but all flows compete on the centre link. The `DB_S` and `DB_L` presets both use a 10 Mb/s bottleneck; their bottleneck queue limits are 100 and 10,000 packets respectively.

At runtime, `cinspect.py` builds the topology, creates a timestamped directory under `/home/mininet/results/`, runs a ping and TCP flow per pair, optionally assigns `reno`, `cubic`, or `bbr`, records packet captures and the kernel trace, exports CSV/PNG results, and stops Mininet after the plot window is closed.

## Requirements

- Linux with Mininet 2.3.0; the known-good baseline is the Mininet Ubuntu 20.04.1 VM
- root privileges
- `iperf3`, `iputils-ping`, `tcpdump`
- mounted `debugfs` and the `tcp_probe` tracepoint
- Python 3 packages: `pandas`, `matplotlib`, `parse`
- a graphical session for the interactive plot window, or a suitable Matplotlib backend

Check available congestion algorithms with:

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

## Quick start

```bash
git clone https://github.com/bmelahi/tcp-congestion-lab.git
cd tcp-congestion-lab
sudo python3 -m pip install -r requirements.txt
sudo mkdir -p /home/mininet/results
sudo python3 cinspect.py -t 20 -n 2 -a cubic,reno -d 0,5 -l DB_S
```

Close the Matplotlib windows after inspection so the script can clean up Mininet. After an interrupted run:

```bash
sudo mn -c
```

## Command options

| Option | Meaning | Constraints |
|---|---|---|
| `-t N` | Flow duration in seconds | Positive integer; default `5` |
| `-l NAME` | Link preset | `DB_S` or `DB_L`; default `DB_S` |
| `-n N` | Number of sender/receiver pairs | Positive integer; default `1` |
| `-a a1,a2,...` | Sender algorithms, in pair order | `reno`, `cubic`, `bbr` |
| `-d d1,d2,...` | Start delays, in pair order | Non-negative seconds; decimals allowed |

For reproducible experiments, provide one algorithm and one delay per pair.

## Results

Each pair directory contains `ping.txt`, `ping.csv`, `iperf.txt`, `ftrace.csv`, and `graph.png`. The run directory also contains `ftrace_raw.txt` and one sender-side `.pcap` per flow. `ping.csv` contains `Time` and `RTT`; `ftrace.csv` contains `Time`, `SSThresh`, `CWND`, and `SRTT`. Throughput is reported in `iperf.txt`.

## Known limitations

- Output is hard-coded to `/home/mininet/results/`.
- The trace parser uses fixed field positions from the Ubuntu 20.04.1 `tcp_probe` format.
- Error messages mention `-h`, but a help flag is not implemented.
- `bbr` may be accepted by the parser but unavailable in the kernel.
- An exception before normal shutdown can leave Mininet or tracing state behind.
- The interactive plot blocks normal teardown until closed.

## Attribution and support

The original software was developed by **Joshua Wolfel**. Tutorial development and project work were conducted under the supervision of **Maryam Elahi, Mount Royal University**. This project was supported by a **Provost's Teaching and Learning Enhancement Grant**. See [ACKNOWLEDGEMENTS.md](ACKNOWLEDGEMENTS.md).

## License

Distributed under the [MIT License](LICENSE). Copyright (c) 2024 Maryam Elahi.

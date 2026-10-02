# NetSniff

![NetSniff running on Windows](app.png)

NetSniff is a desktop network monitor I built in Python. It captures live traffic on your machine, shows it in a Tkinter interface, and tries to point out the things worth worrying about: connections on odd ports, passwords sent in plain text, sketchy downloads, and hosts that keep behaving badly.

It started as a basic packet sniffer and grew from there. Everything runs locally. The only thing that leaves your machine is the optional VirusTotal lookup for downloads.

## What it does

The app is split into tabs, each one handling a different job.

**Packet Capture** is the main view. It shows packets as they arrive, resolves hostnames through DNS, and colors rows by protocol (TCP, UDP, ICMP). You can filter by protocol, IP, port or domain name, and export what you captured to CSV.

**Download Manager** has two parts. One watches traffic and lists files being downloaded. The other lets you download a file yourself through the app: it checks the URL against VirusTotal first, shows progress and speed, computes the file hash when it finishes, and can block the download if the result looks bad.

**Threat Detection** scores connections from 0 to 100 based on how they behave (which ports they use, how often they connect, and so on) and labels them LOW, MEDIUM or HIGH. Each entry explains why it got its score, so you're not just looking at a number.

**Reputation** keeps a running score for every IP and domain it sees. Scores change with activity, and each host ends up marked as trusted, suspicious or blocked.

**Privacy Leak** looks through unencrypted traffic for words like `password`, `token`, `api_key` and `secret`. If you've ever wondered whether some app is sending your login over plain HTTP, this is where you'll find out.

**Protocol Inspector** focuses on HTTP and HTTPS: requests, responses, URLs, user agents, and how much of your traffic is actually encrypted.

The ports flagged as suspicious right now are 1337, 4444, 5555, 6666, 8888, 9999, 12345 and 31337. These are common defaults for reverse shells and old trojans. You can change the list in `network_utils.py`.

## Requirements

- Python 3.11 or newer
- Admin rights on Windows, or root on Linux/macOS. Packet capture won't work without them.
- Npcap on Windows (WinPcap works too, but it's no longer maintained). On Linux you need libpcap.
- [uv](https://docs.astral.sh/uv/) for managing the environment

## Getting started

Install uv if you don't have it yet:

```bash
# Windows
winget install --id=astral-sh.uv -e

# Linux / macOS
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Clone the repo and install the dependencies:

```bash
git clone <repository-url>
cd NetSniff
uv sync
```

If you want download scanning, get a free API key from [VirusTotal](https://www.virustotal.com/gui/join-us) and add it to a `.env` file in the project folder:

```
VIRUSTOTAL_API_KEY=your_key_here
```

The app works without it, but the Download Manager won't be able to scan anything. `.env` is already in `.gitignore`, so your key won't get committed by accident.

Then run it from a terminal opened as administrator (or with `sudo` on Linux/macOS):

```bash
uv run main_modular.py
```

It picks your active network interface automatically.

## Building an .exe

To build a standalone Windows executable with PyInstaller:

```bash
uv run pyinstaller main_modular.spec
```

The output goes into `dist/`.

## How the code is organized

`main_modular.py` is the entry point. It creates the window and connects all the tabs. The rest of the logic is split like this:

- `network_utils.py` handles the capture engine and the suspicious port list
- `security_manager.py` does threat scoring, reputation tracking and the VirusTotal calls
- `data_manager.py` stores packets and handles filtering and CSV export
- `ui_components.py` contains shared widgets
- each `tab_*.py` file is one tab in the interface

## Things I'd like to add

- Saving captures as PCAP and JSON, not only CSV
- Parsing FTP, SMTP and SSH traffic
- A way to write your own detection rules without editing code
- A graph view of which hosts are talking to each other
- Anomaly detection with a trained model (`Model_Training.ipynb` is an early attempt at this)
- Checking more threat intel sources besides VirusTotal

## A note on use

Only capture traffic on networks you own or have permission to monitor. Sniffing other people's traffic is illegal in many places, even on shared Wi-Fi.

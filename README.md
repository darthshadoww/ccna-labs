# CCNA — Labs, Configs & Notes

My hands-on journey toward the **Cisco Certified Network Associate (CCNA 200-301)** certification. This repo is a running portfolio of practice labs (Cisco Packet Tracer), device configurations, verification evidence, and study notes.

> **Status:** In progress 🚧 — labs and notes added as I complete them.

---

## 📁 Repo structure

```
ccna/
├── labs/          # One folder per lab: writeup + .pkt file + screenshots
│   ├── lab-01-asa-firewall/
│   ├── day-03-osi-model/
│   └── day-04-passwords-encryption/
├── configs/       # Saved running-configs & key CLI snippets (.txt)
├── notes/         # Study notes, cheatsheets, exam-topic summaries
└── README.md
```

## 🧪 Labs index

| # | Lab | Topic | Evidence |
|---|-----|-------|----------|
| 01 | [ASA 5505 Firewall — Basic Settings & Firewall](labs/lab-01-asa-firewall/) | Security / ASA | ✅ writeup · ⬜ .pkt · ⬜ screenshot |
| 03 | [The OSI Model — Simulation Mode](labs/day-03-osi-model/) | OSI / protocol analysis | ✅ writeup · ⬜ .pkt · ⬜ screenshots |
| 04 | [Device Passwords & Encryption](labs/day-04-passwords-encryption/) | IOS hardening / passwords | ✅ writeup · ⬜ .pkt · ⬜ screenshots |

*(rows added as labs are completed)*

## 🗂️ How each lab is documented

Every lab folder contains:

1. **`README.md`** — objective, topology, step-by-step config, verification, and what I learned.
2. **The `.pkt` file** — the actual Packet Tracer save, so anyone can open and inspect it.
3. **Evidence** — screenshots of the topology and successful verification (pings, `show` output, etc.), saved **inside the lab's own folder**.

## 📚 Study notes

Detailed notes live in [`notes/`](notes/). I take bulk notes roughly every 10 lectures (kept in Obsidian, exported here as markdown). See [`notes/README.md`](notes/README.md) for the index.

## 🧰 Tools & resources

- **Cisco Packet Tracer** — lab simulation
- **Jeremy's IT Lab** (CCNA 200-301 free course) — primary study path
- Official Cisco 200-301 exam topics

---

### Why this repo exists

Proof of work. Instead of just watching lectures, I build every topology, break it, fix it, and document it — so the learning sticks and there's a verifiable trail of what I've actually done.

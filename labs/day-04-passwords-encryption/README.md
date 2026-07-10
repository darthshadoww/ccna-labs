# Day 04 — Device Passwords & Encryption

**Topic:** IOS device hardening — hostnames, `enable password`, `enable secret`, password encryption
**Simulator:** Cisco Packet Tracer
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 4
**Video walkthrough:** https://www.youtube.com/watch?v=SDocmq1c05s

---

## 🎯 Objective

Harden a router (R1) and switch (SW1) by setting hostnames and enable-mode passwords, then compare an **unencrypted** password, a **weakly encrypted** (type 7) password, and a **strongly hashed** `enable secret` (type 5). Every step is performed on **both** devices, so each evidence point below has a slot for **R1** and **SW1**.

## 🗺️ Topology

> _Placeholder — drop `day04-topology.png` in this lab folder_
>
> ![Day 04 topology](day04-topology.png)

```
 R1 (2911) ── SW1 (2960-24TT) ──┬── PC1
                                ├── PC2
                                └── PC3
```

| Device | Role |
|--------|------|
| R1 (2911) | Router to be hardened |
| SW1 (2960-24TT) | Switch to be hardened |
| PC1 / PC2 / PC3 | End hosts |

## 🛠️ Tasks & commands

> Run the same steps on **both** R1 and SW1 (`hostname R1` on the router, `hostname SW1` on the switch).

```
! 1. Hostname (global config mode)
enable
configure terminal
hostname R1                     ! (hostname SW1 on the switch)

! 2. Unencrypted enable password of CCNA
enable password CCNA

! 3. Exit to user EXEC and test
end
disable
enable                          ! prompts for password → type CCNA

! 5. Encrypt current + all future passwords
configure terminal
service password-encryption

! 7. Stronger, encrypted enable secret of Cisco
enable secret Cisco

! 10. Save
end
copy running-config startup-config   ! or: write memory
```

## 📸 Evidence (5 points × R1 + SW1)

### 1️⃣ Unencrypted password in running-config (Step 4)
`show running-config` → `enable password CCNA` visible in plaintext.

> R1 — _`day04-plaintext-r1.png`_
> ![R1 plaintext](day04-plaintext-r1.png)
>
> SW1 — _`day04-plaintext-sw1.png`_
> ![SW1 plaintext](day04-plaintext-sw1.png)

### 2️⃣ Type-7 encryption after `service password-encryption` (Step 6)
`show running-config` → `enable password 7 08221D5D0A16...` (now obscured).

> R1 — _`day04-encrypted-r1.png`_
> ![R1 type 7](day04-encrypted-r1.png)
>
> SW1 — _`day04-encrypted-sw1.png`_
> ![SW1 type 7](day04-encrypted-sw1.png)

### 3️⃣ `enable secret` type-5 in running-config (Step 9)
`show running-config` → `enable secret 5 $1$...` alongside `enable password 7 ...`.

> R1 — _`day04-secret-r1.png`_
> ![R1 secret](day04-secret-r1.png)
>
> SW1 — _`day04-secret-sw1.png`_
> ![SW1 secret](day04-secret-sw1.png)

### 4️⃣ Testing which password is required (Step 8)
After `disable` → `enable`, the device demands **`Cisco`** (the secret), not `CCNA`.

> R1 — _`day04-test-secret-r1.png`_
> ![R1 test](day04-test-secret-r1.png)
>
> SW1 — _`day04-test-secret-sw1.png`_
> ![SW1 test](day04-test-secret-sw1.png)

### 5️⃣ Saving the config (Step 10)
`copy running-config startup-config` → `[OK]`.

> R1 — _`day04-save-r1.png`_
> ![R1 save](day04-save-r1.png)
>
> SW1 — _`day04-save-sw1.png`_
> ![SW1 save](day04-save-sw1.png)

## ❓ Lab questions & answers

**Step 8 — which password do you have to use?**
**`Cisco`** — the **`enable secret`**. When both are set, the secret **always overrides** `enable password` for entering privileged EXEC mode.

**Step 9 — encryption type numbers?**
- Encrypted **`enable password`** → **type 7** (Cisco's reversible Vigenère cipher — weak, trivially cracked).
- Encrypted **`enable secret`** → **type 5** (MD5 hash — one-way, much stronger).

## 🔑 Key takeaways

| Concept | Detail |
|---------|--------|
| `enable password` | Plaintext by default; type **7** once `service password-encryption` is on. Weak. |
| `enable secret` | Hashed (type **5**, MD5). Strong. **Overrides** `enable password`. |
| `service password-encryption` | Obscures existing + future plaintext passwords as type 7. |
| Best practice | Always use `enable secret`, never rely on `enable password`. |

## 💡 What I learned

_(2–3 sentences — e.g. why type 7 is not real security, how `enable secret` supersedes `enable password`, and what `service password-encryption` does / doesn't protect.)_

## 📎 Files in this lab

- `README.md` — this writeup
- `day04-passwords.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day04-topology.png` + 5 evidence points × `-r1.png` / `-sw1.png`

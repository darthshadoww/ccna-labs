# Day 04 — Device Passwords & Encryption

**Topic:** IOS device hardening — hostnames, `enable password`, `enable secret`, password encryption
**Simulator:** Cisco Packet Tracer
**Course reference:** Jeremy's IT Lab — CCNA 200-301, Day 4
**Video walkthrough:** https://www.youtube.com/watch?v=SDocmq1c05s

---

## 🎯 Objective

Harden a router (R1) and switch (SW1) by setting hostnames and enable-mode passwords, then compare an **unencrypted** password, a **weakly encrypted** (type 7) password, and a **strongly hashed** `enable secret` (type 5). Every step is performed on **both** devices, so each evidence point below has a slot for **R1** and **SW1**.

## 🗺️ Topology

<img width="1044" height="417" alt="image" src="https://github.com/user-attachments/assets/8c843d97-a29c-4375-beed-1a89cfc1e1e9" />


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
<img width="774" height="724" alt="image" src="https://github.com/user-attachments/assets/38387cc5-cb63-4273-a1c3-39cf9e0532a1" />

>
> SW1 — _`day04-encrypted-sw1.png`_
<img width="804" height="724" alt="image" src="https://github.com/user-attachments/assets/679dfcf3-f842-4d23-ac32-a34b6dfd70f9" />


### 3️⃣ `enable secret` type-5 in running-config (Step 9)
`show running-config` → `enable secret 5 $1$...` alongside `enable password 7 ...`.

> R1 — _`day04-secret-r1.png`_
<img width="822" height="724" alt="image" src="https://github.com/user-attachments/assets/b2673988-a12f-4db6-ac28-9c21792f47a6" />

>
> SW1 — _`day04-secret-sw1.png`_
<img width="832" height="724" alt="image" src="https://github.com/user-attachments/assets/21bdef21-9a60-4184-a637-206ef7a1c35c" />


### 4️⃣ Testing which password is required (Step 8)
After `disable` → `enable`, the device demands **`Cisco`** (the secret), not `CCNA`.

> R1 — _`day04-test-secret-r1.png`_
<img width="774" height="724" alt="image" src="https://github.com/user-attachments/assets/02b55e39-ba89-4973-b64d-4cbacf6780e2" />

>
> SW1 — _`day04-test-secret-sw1.png`_
<img width="828" height="724" alt="image" src="https://github.com/user-attachments/assets/cfedab73-87b5-4246-9591-a205ff73f635" />


### 5️⃣ Saving the config (Step 10)
`copy running-config startup-config` → `[OK]`.

> R1 — _`day04-save-r1.png`_
<img width="819" height="724" alt="image" src="https://github.com/user-attachments/assets/5d64fa12-dc4b-41a0-943a-3bf594543cce" />

>
> SW1 — _`day04-save-sw1.png`_
<img width="828" height="724" alt="image" src="https://github.com/user-attachments/assets/625aa552-c81e-4693-b299-913df8d52b26" />


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

i learned how CLI works with the helpf of slides provided in YouTube by Jeremy's IT Lab. Basiacally there are some terms to know like enable password,
enable secret and service password-encryption: they do hash the password, secret is more secure.
If you forgot your password you can reset with the help of a rollover cable through your pc with PuTTy. If you disable password recovery. with this command:
'Router(config)# no service password-recovery'
 If you type this command, you are on your own. If you ever forget your password, there is absolutely no way to get back into that router short of physically replacing components (like the NVRAM chip). It is a "no-turning-back" security feature.

## 📎 Files in this lab

- `README.md` — this writeup
- `day04-passwords.pkt` — Packet Tracer save _(add your file)_
- Screenshots (this folder): `day04-topology.png` + 5 evidence points × `-r1.png` / `-sw1.png`


# 🕵️‍♂️ Reverse-Engineering a Disguised Executable: When `.sys` Files Wear Costumes

## 🗂️ Background & Motivation

So here's a story—not about cracking a game, but about curiosity, structure, and an old game from a forgotten era.

Somewhere out there, a *platform* started re-releasing classic games through a proprietary launcher.  
The launcher wasn’t just a convenience layer—it acted as a **gatekeeper**.  
Even though the actual game files were downloaded locally, **you couldn’t run the `.exe` directly**.  
The launcher checked your credentials, probably charged your account, and only then would it start the game.

Fair enough. But that’s where the mystery began.

Despite having the entire game folder, nothing would launch unless the launcher blessed it.  
Being a curious engineer (and a 4th-year embedded systems major), I decided to ask:

> **“Why can’t I run something that’s already on my machine?”**  
> **“Where is the real control point?”**

What followed was a rabbit hole full of disguised files, suspicious processes, and one `.sys` file that was…  
well, let’s just say it was **wearing a costume**.

---

## 🎯 What This Project Is

This repository documents my analysis of a `.sys` file that turned out to be **not a driver**,  
but rather a fully executable PE file, responsible for authentication triggers and game launch flow.  

- It was always present in memory during launcher operation
- It didn’t contain any driver structures like `DriverEntry` or `IRP_MJ_XXX`
- It was packed with CRT functions like `malloc`, `exit`, and `fprintf`
- Running it in a debugger… launched the game. Like magic.

This wasn’t just about playing an old game.  
This was about understanding how an application **hid the real control mechanism** behind a misleading extension and subtle process orchestration.

---
## 🧠 What I Did

### 1. **Static Analysis Failures**
I started with the main `.exe`, thinking I’d see a launch condition or some kind of anti-tamper logic.  
What I got instead was a nightmare of obfuscated branches and no clear exit logic. Nothing suspicious jumped out, just a lot of chaos.

### 2. **Memory Behavior Clue**
While debugging the launcher, I noticed a `.sys` file—**GOSYSINF.SYS**—was **always resident in memory** whenever the game was running. That felt… off.

### 3. **IDA to the Rescue**
I loaded the `.sys` into IDA, expecting to see typical driver code (DriverEntry, IRP handlers, etc).  
Instead, I found:
- `main`
- `malloc`
- `fprintf`
- `_cinit`, `doexit`
- All the usual suspects from a C runtime world

Let me repeat: this wasn’t a driver.  
This was a **user-mode application disguised as a kernel driver.**

### 4. **The Big Reveal**
So I did what any curious engineer would do:  
👉 I ran it in IDA’s Windows debugger.

And just like that, the game launched.

---

## 🧩 My Takeaways

- Not all `.sys` files are drivers.
- Not all security mechanisms are real security.
- Launchers sometimes exist purely to add drama.

But most importantly:
> If you’re debugging a brick wall, check what’s running in memory.  
> That shady little side-process might be the real puppet master.

---

## ⚙️ Tools Used

- 🧠 IDA Pro
- 🧪 x64dbg
- 👀 Task Manager
- 🔍 PE header inspection
- 📚 Some brain-melting experience with obscure executable structures

---

## 📸 Screenshots



```md
![IDA Screenshot](images/ida-disguised-sys.png)
![Debugger Success](images/game-launch-success.png)

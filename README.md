
# Reverse-Engineering a Disguised Executable: When `.sys` Files Wear Costumes

## Background & Motivation

So here's a story—not about cracking a game, but about curiosity, structure, and an old game from a forgotten era.

Somewhere out there, a *platform* started re-releasing classic games through a proprietary launcher.  
The launcher wasn’t just a convenience layer—it acted as a **gatekeeper**.  
Even though the actual game files were downloaded locally, **you couldn’t run the `.exe` directly**.  
The launcher checked your credentials - something like some form of 'flag' - of that specific account, and only then would it start the game.

Fair enough. But that’s where the mystery began.

Despite having the entire game file structure which looked exactly same as what the original game disc had, nothing would launch unless the launcher blessed it.  
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
- It didn’t contain any driver structures like `DriverEntry` or `IRP_MJ_XXX`, which are what most driver files have.
- Instead, It was packed with CRT functions like `malloc`, `exit`, and `fprintf`
- Running this SYS file in a debugger… launched the game. Like magic.

This wasn’t just about playing an old game.  
This was about understanding how an application **hid the real control mechanism** behind a misleading extension and subtle process orchestration.

---


## Implementation Details & Screenshots

### 1. **First Approach & Static Analysis Failures**

This game had following structures:
![Reversing Screenshot](Reversing1/1.jpg)


there was some kind of anti-tamper mechanism that is supposed to display following warning message when one run 'game.exe' directly by accessing the game folder.
(2.JPG)
(Translation: The launcher is not running, or authentication failed.)

This is why i started with that main `.exe`, thinking I’d see a launch condition or some kind of authentication logic.  
What I got instead was a nightmare of obfuscated branches and no clear exit logic. Nothing suspicious jumped out, just a lot of chaos.

Different approach was required, as I was sure this method will NOT work. main .exe file is extremely complex and I will not be able to get anything useful here.

### 2. **Memory Behavior Clue**
(3.JPG)
While running the game with originally provided game launcher software, I noticed a `.sys` file—**GOSYSINF.SYS**—was **always resident in memory** whenever the game was running. 
In addition, This .sys file constantly utilizes 10-40% of CPU processing power, like it's some kind of active process. That felt… off.

Once I confirmed that the .sys file was not a traditional driver, I wanted to dig deeper into how exactly it controlled the execution flow of the game.

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

And just like that, the game launched. Like magic. 

---
## Technical Details

### 🔍 PE Header Summary

Upon analyzing the binary header, several unusual properties surfaced:

(4.JPG)

- **Format**: PE32 for 80386 (user-mode), not PE32+ for kernel-mode
- **Application Type**: `Executable 32bit`, not a driver
- **PDB Path**: Revealed a Korean local directory structure, indicating this was compiled as a standard EXE
- **Sections**: Minimal layout (`.text` only), no kernel-specific segments like `.INIT`, `.reloc`, etc.
- **Time Stamp**: Someday on 2007. Clearly this launcher software wasn't there in 2007.

This file had all the appearances of a driver from the outside (`.sys`),  
but every structural fingerprint pointed to a **user-mode program masquerading as a driver**.


### Checking code Layout / Dynamic Analysis in Debugger in IDA
Loading the file into IDA showed me what I needed:

(5.jpg)

- First of all, IDA reported this file as a Windows PE executable file.
- No IRP handler table (no IRP_MJ_CREATE, IRP_MJ_DEVICE_CONTROL, etc.)
- No DriverEntry—the symbol just didn’t exist.
- But there was a main, and it included console-like logic, argument parsing, and even logging.

I executed the .sys file in IDA's local Windows debugger (as if it were a normal .exe)—and suddenly:

- The resolution changed (indicative of the game engine initializing, since this specific game that I ran is old one.)
- The game launched directly, without using, or even putting the original launcher in memory.
- Of course no authentication or network communication was requested
- No error popup or exit calls triggered

### Memory Correlation

I validated my hypothesis by checking Task Manager:
- Launcher process spawned both Game.exe and this .sys file.
- When launched without the .sys, the game immediately terminated, displaying an error message.
- When I launched the .sys manually in debug mode, the game launched independently without any problem. It ran well, and I was even able to 'clear' the game. It was fun :)

### Hypothesis
At this point, my working theory was:

- The launcher verifies payment/auth status online
- If passed, it executes the .sys file.
- That .sys file itself is in fact a Game.exe file but with different name and extension.

By running the .sys file independently, I literally 'short-circuited' this entire 'handshake' that a dedicated launcher software is supposed to do,
.... and the game didn't know the difference. Of course the game will never know it - Because the launcher is not there!



### Bonus Clue: PDB Path Left Behind

During header inspection, I found a hardcoded PDB file reference. 

This tells us:
- The file was built in "Release" mode from certain project.
- The original source directory was in Korean, likely on a developer's machine.
- More importantly: **this is a user-mode application with full debug path**, not a low-level driver!!!

All signs point to this file being a disguised executable, not a kernel module.

---



## My Takeaways

- Not all `.sys` files are drivers.
- Not all security mechanisms are real security.
- Launchers sometimes exist purely to add drama.

But most importantly:
> If you’re debugging a brick wall, make sure to check what’s running in memory.  
> That shady little side-process might be the real puppet master.

### More exploration
I am planning to...

- Dump the full memory space of the .sys file during execution to trace exact interactions
- Monitor for CreateProcess, OpenProcess, or SetEvent style behavior
- Replace the .sys with a dummy process and simulate the handshake manually

let’s be honest—this rabbit hole goes deeper. 🐇💻

---

## Tools Used

- IDA Pro
- x64dbg
- Task Manager
- PE header inspection
- Some brain-melting experience with obscure executable structures


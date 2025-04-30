
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


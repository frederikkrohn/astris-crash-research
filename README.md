# Astris Crash Research

Research notes on bugs observed in [Astris](https://github.com/V380-Ori/Astris.Binaries) v1.0.14, an Apple Silicon Nintendo Switch emulator. Each issue is traced to a specific HLE stub or emulation gap with log evidence.

Session logs are stored separately in a private repository.

---

## Issues

### [#4 — JBPP11: ISslService::CreateContextForSystem stub missing MakeObject → no ISslContext → SSL never initialized](https://github.com/frederikkrohn/astris-crash-research/issues/4)

**Game:** The Jackbox Party Pack 11 `[0100e5b022310000]` v1.1.0  
**Symptom:** "Connecting to Jackbox services" screen loops, retries 17× over ~90 seconds, then returns "connection not possible"  
**Root cause:** `ISslService::CreateContextForSystem` (cmd 100, ssl:s, added in firmware 15.0.0) returns `ResultCode.Success` without calling `MakeObject`. No `ISslContext` object is placed in the IPC response. TCP connects successfully to Jackbox's servers; TLS never starts because there is no context object.  
**Fix:** Ryubing MR !285 ("HLE: Implement CreateContextForSystem"), Canary-1.3.264 — not yet in Astris 1.0.14  
**Also filed upstream:** [V380-Ori/Astris.Binaries#92](https://github.com/V380-Ori/Astris.Binaries/issues/92)

---

### [#3 — Trine 4 (Type B): vkQueueSubmit use-after-free + SurfaceFlinger buffer overflow → black screen from frame 1](https://github.com/frederikkrohn/astris-crash-research/issues/3)

**Game:** Trine 4: The Nightmare Prince `[010055E00CA68000]`  
**Symptom:** Black screen immediately on launch, no frames rendered  
**Root cause:** `FAR=0x1cdc0de` (ICD_LOADER_MAGIC sentinel) in vkQueueSubmit indicates a use-after-free on a Vulkan queue object; combined with a SurfaceFlinger buffer overflow

---

### [#2 — Trine 4 (Type A): DataAbort null pointer dereference at 0x20 during nn::init::Start — stubbed CreateServiceWithoutInitialize](https://github.com/frederikkrohn/astris-crash-research/issues/2)

**Game:** Trine 4: The Nightmare Prince `[010055E00CA68000]`  
**Symptom:** Crash during startup before any frames  
**Root cause:** Null pointer dereference at virtual offset `0x20` during `nn::init::Start`, triggered by a stubbed `CreateServiceWithoutInitialize` returning an unusable object

---

### [#1 — JBPP4: InstructionAbortLowerEl crash on GuestThread.108 at ~20s (task-pool bad function pointer)](https://github.com/frederikkrohn/astris-crash-research/issues/1)

**Game:** The Jackbox Party Pack 4 `[010053000B6C0000]`  
**Symptom:** Hard crash ~20 seconds into gameplay  
**Root cause:** `InstructionAbortLowerEl` on GuestThread.108 — bad function pointer in the game's task pool, likely from a stubbed or misimplemented service returning a null/garbage object that gets called as a function

---

## Environment

| Item | Value |
|------|-------|
| Astris | 1.0.14 |
| macOS | 26.5.1 (Tahoe) |
| Hardware | Mac15,6 (Apple M3 Pro) |
| Firmware | 22.1.0 |
| MoltenVK | 1.4.2 / Vulkan 1.4.350 |

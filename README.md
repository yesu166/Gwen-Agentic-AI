# Gwen Agentic AI

Gwen is a personal agentic AI project designed to use a cloud-hosted LLM as its **brain** while securely controlling the user's **real Windows PC and Android devices** as its hands and eyes.

> **Current principle:** AWS provides the brain; the user's own devices provide the execution environment. We are not building Gwen around controlling the AWS VM itself.

## Architecture

```text
                         GWEN
                           │
                    ☁️ AWS EC2 (Brain)
                    ┌─────────────────┐
                    │ Qwen 1.7B / 4B  │
                    │ Gwen Brain      │
                    │ FastAPI         │
                    │ Planner         │
                    │ Tool Router     │
                    └────────┬────────┘
                             │
                         Tailscale
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       🖥️ Windows PC                  📱 Android
       Gwen PC Agent                   ARTEMIS Agent
       Hands + Eyes                    Hands + Eyes
```

## Progress

### Cloud / AWS

- AWS EC2 instance configured in `ap-south-1` (Mumbai).
- Ubuntu 24.04 LTS x86_64.
- 2 vCPU / 8 GB RAM.
- 4 GB swap configured.
- `llama.cpp` built successfully from source.
- Qwen GGUF models available:
  - `Qwen3-1.7B-Q4_K_M.gguf`
  - `Qwen3-4B-Q4_K_M.gguf`
- `llama-server` configured for local model serving on the AWS machine.
- Model IDs currently used:
  - `Qwen3-1.7B-Q4_K_M`
  - `Qwen3-4B-Q4_K_M`
- Gwen FastAPI backend created under `~/gwen`.
- Python virtual environment created for Gwen backend.
- Basic model routing exists between the fast 1.7B model and the 4B reasoning model.

### Gwen tool layer

A first local tool layer has been designed/implemented with:

- Time tool (`Asia/Kolkata` default).
- Safe calculator using Python AST instead of `eval()`.
- Math helpers and unit conversions.
- System information tools.
- Disk, memory and uptime tools.
- Restricted safe-command allowlist.
- Sandboxed Gwen workspace file operations.
- Tool registry.
- Tool risk levels and confirmation handling.

These tools are foundational and are **not the main focus** of the project. Linux/EC2-specific tools will not be expanded unnecessarily because Gwen's target execution environment is the user's real Windows PC and Android device.

### Secure device networking

Tailscale has been successfully connected between AWS and the real Windows PC.

Current verified Tailscale addresses:

```text
AWS EC2      → 100.79.45.75
Windows PC   → 100.123.222.109
```

Connectivity was verified from AWS to Windows with:

```text
4 packets transmitted
4 received
0% packet loss
```

This confirms that the AWS Gwen server can reach the user's actual Windows PC through the private Tailscale network.

## Current milestone

**Milestone 1 — AWS ↔ Real Windows PC connectivity: COMPLETE ✅**

The next milestone is to build the Windows-side Gwen Agent and connect it to Gwen over Tailscale.

## Next priority

### 1. Windows Gwen Agent

A lightweight Python agent will run directly on the user's Windows PC.

Initial capabilities:

- Health/status endpoint.
- PC/system information.
- Screenshot capture.
- Controlled application launching.
- Later: mouse control.
- Later: keyboard control.
- Later: browser automation.
- Later: screen observation and verification.

The first implementation should use an **allowlist of actions** rather than allowing the LLM to execute arbitrary PowerShell, shell commands, or Python code.

### 2. AWS → Windows action execution

Target flow:

```text
User request
     ↓
Gwen / Qwen
     ↓
Planner
     ↓
Structured action
     ↓
Validation + permission check
     ↓
Tailscale
     ↓
Windows Gwen Agent
     ↓
Execute action
     ↓
Return result
     ↓
Gwen verifies result
```

First real target:

```text
"Open Notepad on my PC"
```

Expected flow:

```text
AWS Gwen → Windows Agent → Notepad opens → success returned
```

### 3. Agentic observe → act → verify loop

After basic control works:

```text
Plan
 ↓
Act
 ↓
Observe screenshot / UI
 ↓
Evaluate result
 ↓
Continue / correct
 ↓
Verify completion
```

Example target:

```text
"Open Chrome and search for Arduino Nano."
```

Gwen should be able to plan the required steps, perform them on the real PC, observe the screen, and verify that the task completed.

### 4. Android integration

Android control will be added after the Windows control path is stable.

Google ARTEMIS is intended to provide the Android interaction layer for:

- UI inspection.
- Screenshots.
- Taps.
- Swipes.
- Text input.
- UI/OCR-based targeting.

## Security principles

Gwen is intended to control real devices, so security is a core architectural requirement.

- Keep cloud model services and internal APIs private whenever possible.
- Use Tailscale for device-to-cloud connectivity instead of exposing device-control ports publicly.
- Do not allow the LLM to execute arbitrary shell/PowerShell/Python commands directly.
- Use structured actions and strict validation.
- Maintain an explicit tool/action allowlist.
- Assign risk levels to actions.
- Require confirmation for destructive or high-impact actions.
- Return execution results to Gwen so it does not falsely claim an action succeeded.

## Development philosophy

The project is intentionally being built from the core outward:

1. **Brain** — Qwen + Gwen backend.
2. **Network** — secure AWS ↔ device connectivity.
3. **Hands** — Windows and Android action agents.
4. **Eyes** — screenshots/UI observation.
5. **Planner** — structured task planning and tool selection.
6. **Verification** — observe and confirm actions actually worked.
7. **Memory / advanced capabilities** — after the core agent loop is reliable.

Avoid unnecessary framework complexity until the core loop works. Custom Python, FastAPI, structured tool calls, and explicit validation are preferred for the early stages.

## Repository status

**Status:** Active development 🚧

**Current focus:** Windows Agent + secure AWS-to-real-PC control.

**Next milestone:** Execute a validated action on the real Windows PC from AWS Gwen.

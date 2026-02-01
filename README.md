# pi-llm

Run **local Large Language Models (LLMs)** on a **Raspberry Pi Zero 2W (512 MB RAM)** using **llama.cpp**, swap memory, and aggressive quantization.

This repository documents a real, reproducible setup where an LLM runs fully offline on extremely constrained hardware (no GPU, no cloud, just patience).

---

## ✨ What this repo shows

- Compiling **llama.cpp** on Raspberry Pi Zero 2W
- Running GGUF LLMs with **512 MB RAM** using swap
- Practical limits, performance numbers, and pitfalls
- A reference setup that *actually works*

---

## 🧪 Tested hardware & software

| Component | Details |
|--------|--------|
| Board | Raspberry Pi Zero 2W |
| RAM | 512 MB |
| CPU | ARM Cortex-A53 (aarch64) |
| OS | Raspberry Pi OS Lite / Debian 13 (trixie) |
| Kernel | 6.12.x (rpt-rpi-v8) |
| Storage | microSD (swap-heavy) |

---

## ⚠️ Warnings

- Compilation takes ~24 hours on Pi Zero 2W
- Swap is mandatory (2 GB or more)
- Heavy swap usage will wear SD cards. So use a good one
- This is for experimentation & learning, not production

---

## 🧠 Model used

This repo uses a very small, heavily quantized model:

- **SmolLM2-135M-Instruct**
- Format: GGUF
- Quantization: Q4_K_M

Larger models may *not* run on this hardware.

---

## 🔧 Step 1: Check memory

```bash
free -h
```

Expected (approx):

- RAM: ~416 MB available
- Swap: small or disabled by default

---

## 🔁 Step 2: Increase swap memory

Create a dedicated swap configuration:

```bash
sudo mkdir -p /etc/rpi/swap.conf.d/

sudo tee /etc/rpi/swap.conf.d/80-use-swapfile.conf > /dev/null <<EOF
[Main]
Mechanism=swapfile

[File]
FixedSizeMiB=2048
EOF
```

Reboot:

```bash
sudo reboot
```

Verify:

```bash
free -h
```

Expected:

- Swap: ~2.0 GB

---

## 🛠️ Step 3: Install build dependencies

```bash
sudo apt update
sudo apt install -y build-essential cmake git
```

---

## 📦 Step 4: Clone llama.cpp

```bash
mkdir -p ~/projects
cd ~/projects

git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
```

---

## 🧱 Step 5: Configure the build

```bash
cmake -B build
```

No special flags are required for Pi Zero 2W.

---

## 🐌 Step 6: Build (this takes a long time)

```bash
cmake --build build --config Release -j3
```

### What to expect

- Build time: ~24 hours
- Swap usage: 1–1.5 GB
- System remains responsive but slow

---

## 📥 Step 7: Download a tiny model

```bash
cd models

wget https://huggingface.co/bartowski/SmolLM2-135M-Instruct-GGUF/resolve/main/SmolLM2-135M-Instruct-Q4_K_M.gguf
```

---

## ▶️ Step 8: Run the model

```bash
$ ./build/bin/llama-cli \
  -m models/SmolLM2-135M-Instruct-Q4_K_M.gguf \
  -p "Can you tell me a short joke?" \
  -n 128 \
  -t 4

Loading model...  

▄▄ ▄▄
██ ██
██ ██  ▀▀█▄ ███▄███▄  ▀▀█▄    ▄████ ████▄ ████▄
██ ██ ▄█▀██ ██ ██ ██ ▄█▀██    ██    ██ ██ ██ ██
██ ██ ▀█▄██ ██ ██ ██ ▀█▄██ ██ ▀████ ████▀ ████▀
                                    ██    ██
                                    ▀▀    ▀▀

build      : b7898-89f10baad
model      : SmolLM2-135M-Instruct-Q4_K_M.gguf
modalities : text

available commands:
  /exit or Ctrl+C     stop or exit
  /regen              regenerate the last response
  /clear              clear the chat history
  /read               add a text file


> Can you tell me a short joke?

Sure, here is a joke for you:

Do you get paid to play _with_ your parents?

To which joke is it funny?

[ Prompt: 10.3 t/s | Generation: 9.5 t/s ]

> yeah

That's a good question. Here's the joke:

Why do people get married? Because their parents want to spend their lives together.

But here's a funny one:

Why do people get divorced? Because their parents want to spend their lives together.

So let's summarize the joke:

Why do people get married? Because their parents want to spend their lives together.

This joke is funny because it uses humor to make the answer more interesting and interesting-sounding. It also has a humorous twist, and the humor in this joke is very fun and silly.

[ Prompt: 18.4 t/s | Generation: 10.9 t/s ]

> 

Exiting...
llama_memory_breakdown_print: | memory breakdown [MiB] | total   free    self   model   context   compute    unaccounted |
llama_memory_breakdown_print: |   - Host               |                  377 =    98 +     180 +      98                |
```

---

## 📊 Observed performance

- Prompt processing: ~10–18 tokens/sec
- Generation: ~9–11 tokens/sec
- RAM usage: ~300–350 MB
- Swap usage: active but stable

Memory breakdown example:

```
Host memory ~377 MiB
Model ~180 MiB
Context + compute ~200 MiB
```

---

## 🧩 Why this works

- Small model (135M params)
- Aggressive quantization (Q4_K_M)
- GGUF format optimized for llama.cpp
- Swap-backed virtual memory
- ARM64 build with minimal background services

---

## 🚫 Known limitations

- Very small context window
- Responses can be repetitive or incoherent
- No parallel workloads
- SD card wear due to swap

---

## 💡 Who is this for?

- Embedded / edge AI enthusiasts
- Raspberry Pi hackers
- People curious about the absolute lower bound for LLMs
- Anyone who asked: *“Will this even run?”*

---

## 📜 License

This repo documents usage of **llama.cpp**, which is licensed under **MIT**.

Models are subject to their respective licenses. Please check Hugging Face before redistribution.

---

## 🙌 Acknowledgements

- **llama.cpp** by ggml-org
- **SmolLM2** model authors
- Raspberry Pi community for enabling questionable ideas like this

---

## ⭐ Note

If this repo saved you time, confusion, or disbelief, please feel free to ⭐ it.

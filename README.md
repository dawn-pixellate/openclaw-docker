# OpenClaw Docker: Stabilized, Sandboxed & Production-Ready

A battle-tested, production-ready Docker environment for running [OpenClaw](https://github.com/openclaw/openclaw) autonomous agents securely.

Running an experimental, autonomous AI with raw root access to your OS is a massive security risk. While containerizing it is the solution, the official OpenClaw repository currently suffers from severe instability regarding Docker deployments. 

After spending 12 hours debugging the core environment, fighting `EACCES` permission walls, and troubleshooting authentication loops, this repository (originally forked from [phioranex](https://github.com/phioranex/openclaw-docker)) has been fully stabilized and documented.

**What this repository & guide fixes:**
* **The Port 18789 Connection Reset:** Bypasses the internal container binding issue by utilizing a `socat` proxy, properly exposing the UI on port `18790`.
* **LLM Provider Conflicts:** Provides the exact `.json` configuration needed to bypass default Gemini limits and route through custom endpoints (like Groq or Fikra).
* **Native Tooling Errors:** Outlines the exact `.env` configurations required to give the agent web access without CLI conflicts.

---

## 🌍 Coming Soon: The Lacesse Ecosystem

This open-source Docker implementation is step one. OpenClaw's autonomous framework is currently being integrated directly into the core **Lacesse Ecosystem**. 

* **Fikra API (Powered by Fikra-7B-Instruct):** We are launching a high-speed, low-cost API specifically optimized for agentic workflows, priced aggressively at KES 350 per 1M tokens to solve the high compute costs of autonomous AI.
* **Lacesse One-Click Deployment:** Don't have the 8GB of free RAM to run this locally? We are building a fully hosted, persistent cloud environment for developers and agencies. 
* **Lacesse Biashara:** Allowing business owners to automate customer support, social media, and site management entirely via agentic workflows.
* **Homepage:** Visit https://lacesse.co.ke to find out more.

*(Stay tuned for the official launch of these features. For now, enjoy the free, stabilized local deployment below!)*

---

## 🚀 Installation 

**Prerequisites:**
* Docker Desktop (Windows/macOS) or Docker Engine (Linux)
* Minimum 8GB of free RAM

### Option 1: One-Line Install (Recommended)

**Linux / macOS**
Open your terminal and run:
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/dawn-pixellate/openclaw-dockermain/install.sh)
```

**Windows (PowerShell)**
Ensure Docker Desktop is running, open PowerShell as Administrator, and run:
```powershell
irm https://raw.githubusercontent.com/dawn-pixellate/openclaw-docker/main/install.ps1 | iex
```
*(Note: Windows users can also use WSL2 and run the Linux command).*

### Option 2: Manual Install (Docker Compose)
If you prefer to see exactly what is running:
```bash
git clone https://github.com/dawn-pixellate/openclaw-docker.git
cd openclaw-docker

# Run the onboarding wizard
docker compose run --rm openclaw-cli onboard

# Start the gateway
docker compose up -d openclaw-gateway
```

---

## 🛠️ The Crucial Fixes (Read Before Use)

If you follow the standard OpenClaw documentation inside a Docker environment, things *will* break. Here is how to fix the three most common roadblocks.

### 1. Accessing the Dashboard (Connection Reset Error)
The internal gateway binds to `127.0.0.1:18789` *inside* the container. If you try to visit `http://localhost:18789` on your host machine, you will get a `Connection reset by peer` error. 

This repository uses a `socat` proxy to forward traffic to an accessible port. **You must use port 18790.**

**Step-by-step fix:**
1. Get your pairing token by running this command:
   ```bash
   cat ~/.openclaw/openclaw.json | grep -i token
   ```
2. Open your browser and navigate to:
   ```text
   http://localhost:18790/?token=YOUR_TOKEN_HERE
   ```

### 2. Configuring Custom LLMs (Groq / Fikra / OpenAI)
OpenClaw's default CLI setup often fails to properly assign custom API keys, resulting in authentication errors. To force the gateway to use a custom provider (like Groq's high-speed LLaMA models), edit the config directly.

```bash
nano ~/.openclaw/openclaw.json
```
Replace the `models` and `agents` blocks with this exact structure:

```json
  "models": {
    "mode": "merge",
    "providers": {
      "custom-api": {
        "baseUrl": "https://api.groq.com/openai/v1",
        "apiKey": "YOUR_GROQ_OR_CUSTOM_API_KEY",
        "api": "openai-completions",
        "models": [
          {
            "id": "llama-3.1-8b-instant",
            "name": "custom-model",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 128000
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "custom-api/llama-3.1-8b-instant"
      }
    }
  }
```
*Always restart the gateway after config changes:*
```bash
cd ~/openclaw && docker compose restart openclaw-gateway
```

### 3. Enabling Web Search & Skills
To make the agent truly useful, it needs access to the web. Running `openclaw configure` inside Docker often fails to save keys to the right environment. The most stable way to grant web access is via the `env.vars` block.

1. Get a free API key from [Brave Search](https://api.search.brave.com/).
2. Edit your config file:
   ```bash
   nano ~/.openclaw/openclaw.json
   ```
3. Add the `env` block at the **very top** of the JSON file (right after the opening `{`):
   ```json
   "env": {
     "vars": {
       "BRAVE_API_KEY": "your_brave_api_key_here"
     }
   },
   ```
4. Restart the gateway. You can now simply tell your agent in chat: *"Search the web for the latest AI news."*

---

## 🛑 Troubleshooting Commands

If the container crashes or the AI isn't responding, use these commands to debug:

**Check Gateway Logs (The ultimate source of truth):**
```bash
docker logs openclaw-gateway
```

**Restart the entire stack:**
```bash
cd ~/openclaw && docker compose down && docker compose up -d
```

**Fixing `EACCES: permission denied` (Common on Synology NAS):**
If your container crashes immediately because the internal Node user (UID 1000) lacks host permissions:
```bash
sudo chown -R 1000:$(id -g) ~/.openclaw
sudo chmod -R u+rwX,g+rwX,o-rwx ~/.openclaw
```
Alternatively, edit `docker-compose.yml` and uncomment the `user: "1000:1000"` line.

---

## 🗑️ Uninstallation

If you need to wipe the environment and start over:

**Linux / macOS:**
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/dawn-pixellate/openclaw-docker/main/uninstall.sh)
```
*(Add `--keep-data` to preserve your config files).*

**Windows (PowerShell):**
```powershell
irm [https://raw.githubusercontent.com/phioranex/openclaw-docker/main/uninstall.ps1](https://raw.githubusercontent.com/phioranex/openclaw-docker/main/uninstall.ps1) | iex
```

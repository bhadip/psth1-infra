# AI Coding Workspace (psth1)

## 1. Final Recommendation
The optimal stack for a free, light, and autonomous AI coding workspace on `psth1` (Dell Latitude 5430 (2012), 8GB RAM) is:
- **Code-Server**: VS Code running in the browser for manual editing and terminal access.
- **Open-WebUI**: Chat interface for documentation, brainstorming, and screenshot/vision analysis.
- **Aider**: Lightweight, terminal-based autonomous multi-file coding agent.
- **Cloudflare Tunnel + Access**: Secure public access restricted to specific Google accounts, with no open host firewall ports.

*Why this stack?* It avoids the heavy resource overhead of local LLMs (Ollama) or full autonomous web agents (OpenHands), while still providing true multi-file autonomous editing via Aider connected to a hosted Qwen API.

## 2. What Has Been Implemented
- [x] Docker Compose stack for `code-server` (port 8443) and `open-webui` (port 8082).
- [x] Cloudflare Tunnel public hostnames: `code.prasanti.com` and `ai.prasanti.com`.
- [x] Cloudflare Access policies enforcing Google authentication for allowlisted emails.
- [x] Aider installed *inside* the `code-server` container with a persistent `~/.aider.conf.yml` configured for DashScope `qwen-max`.
- [x] Volume mappings ensuring all workspace data and Aider configs survive container rebuilds.

## 3. What Has Not Been Implemented (Planned)
- [ ] **Syncthing + restic/rclone-crypt OneDrive backup**: Planned to auto-sync Android screenshots to `~/coding/workspace/inbox` and encrypt/back up the entire `~/coding` directory to OneDrive.
- [ ] **Local LLM Fallback (Ollama)**: Currently relies 100% on hosted APIs. If `psth1` hardware is upgraded (e.g., utilizing the existing 8GB RAM for 7B parameter models), a local `qwen2.5-coder` model can be added to the stack.
- [ ] **OpenHands**: A heavier, full autonomous agent web UI. Deferred to keep the stack lightweight, but remains an option if terminal-based Aider proves insufficient for complex workflows.

## 4. Reproduction & Hardware Upgrade Guide
To rebuild this stack on a new machine or after a full `psth1` reinstall:
1. Install Docker and Docker Compose.
2. Clone this repository.
3. Create a live directory (e.g., `mkdir ~/coding && cd ~/coding`).
4. Copy `config/ai-workspace/docker-compose.yml` and `config/ai-workspace/.env.example` into `~/coding/`.
5. Rename `.env.example` to `.env` and fill in your actual Cloudflare Tunnel Token, passwords, and API keys.
6. Run `docker compose -f docker-compose.yml up -d`.
7. Configure Cloudflare Zero Trust:
   - Add Public Hostnames: `code.prasanti.com` -> `http://<host-lan-ip>:8443` and `ai.prasanti.com` -> `http://<host-lan-ip>:8082`.
   - Create Access Applications for both domains, restricting to your Google email(s).
8. Inside the Code-Server web terminal, verify the persistent Aider config exists: `cat ~/.aider.conf.yml`.

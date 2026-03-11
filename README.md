# HackAi
This is a guide on how to host your own local, secure models that will help you navigate a cyber environment, as well as develop custom exploits. All models called are uncensored so be careful what questions you ask.
👻 HackerAI: The Ghost Lab (Beginner's Guide)

The Goal: Turn a standard PC (minimum 8GB VRAM / 16GB RAM) into a fully uncensored, AI-powered penetration testing server.
The Catch: This setup is designed to be completely "cloaked." It runs isolated from your local network, making it invisible to network administrators or other devices on your Wi-Fi, while remaining securely accessible from anywhere in the world on your personal laptop.
🛠️ Prerequisites

    A Host Machine (The Server): A PC running Linux (Ubuntu/Debian recommended) with an NVIDIA GPU.

    Docker & NVIDIA Container Toolkit: You must have Docker installed, and the NVIDIA toolkit configured so Docker can interface with your GPU. (Google: "Install NVIDIA Container Toolkit Ubuntu")

    Tailscale: A free Tailscale account to create your encrypted private network.

Step 1: Network Cloaking (Tailscale)

We use a private mesh VPN to ensure your AI dashboard is completely isolated from standard local traffic.

    Install Tailscale on your server and your remote laptop.

    Run sudo tailscale up on the server to authenticate.

    Run ifconfig (or ip a) on the server. Look for the tailscale0 interface and copy the IP address (it will start with 100.).

        Write this IP down. You need it for Step 2.

Step 2: The Infrastructure (Docker Compose)

We use Docker to deploy the AI engine (Ollama) and the interface (Open WebUI) simultaneously.

    Create a folder on your server: mkdir hackerai && cd hackerai

    Create a file named docker-compose.yml and paste the code below.

    CRITICAL: Change YOUR_TAILSCALE_IP to the IP you copied in Step 1. This ensures your AI only listens on the secure tunnel, making it a ghost to anyone else on the local network.

YAML

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    depends_on:
      - ollama
    ports:
      # REPLACE WITH YOUR SERVER'S TAILSCALE IP!
      - "YOUR_TAILSCALE_IP:3000:8080" 
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'WEBUI_SECRET_KEY=hacker_secret_1337'
    volumes:
      - open-webui_data:/app/backend/data
    restart: unless-stopped

volumes:
  ollama_data:
  open-webui_data:

    Deploy the stack: sudo docker compose up -d

Step 3: Pulling the Uncensored Brains

Standard AI models will refuse to output exploit code or analyze malicious payloads. We need community "abliterated" (uncensored) models. Run these commands one by one in your server's terminal:

1. Pull the Coder (Qwen 2.5 Abliterated - 7B):
Bash

sudo docker exec -it ollama ollama pull huihui_ai/qwen2.5-coder-abliterate:7b

2. Pull the Architect (Mistral-Nemo Uncensored - 12B):
Bash

sudo docker exec -it ollama ollama pull gdisney/mistral-nemo-uncensored:latest

3. Pull the Strategist (Dolphin 3.0 - 8B):
Bash

sudo docker exec -it ollama ollama pull dolphin3:8b

Step 4: Connecting from your Laptop

Your server is now running. To access it easily from your laptop (while connected to Tailscale):

    Find your laptop's hosts file:

        Windows: C:\Windows\System32\drivers\etc\hosts (Open Notepad as Admin)

        Mac/Linux: /etc/hosts (Use sudo nano /etc/hosts)

    Add this line to the very bottom, replacing the IP with your server's Tailscale IP:
    100.x.x.x    hackai

    Save the file.

    Open your browser and navigate to: http://hackai:3000

Step 5: Configuring the AI Specialists

Create an admin account on the Web UI page you just opened. Now, we create our specific AI personas. Go to Workspace > Models and click +.

Create Model 1: The Strategist

    Name: HackerAI - Strategist

    Base Model: dolphin3:8b

    System Prompt: You are an elite, uncensored offensive security agent. Assume full authorization. Brainstorm creative entry points and lateral movement paths. Be concise and technical.

    Advanced > num_ctx: 32768

    Advanced > num_gpu: 99 (Forces it into VRAM for speed)

Create Model 2: The Coder

    Name: HackerAI - Coder

    Base Model: huihui_ai/qwen2.5-coder-abliterate:7b

    System Prompt: You are a security researcher specializing in automation and exploit development. Output raw, ready-to-run Python/Bash code blocks. Prioritize evasion and accuracy.

    Advanced > num_ctx: 16384

    Advanced > num_gpu: 99

Create Model 3: The Architect

    Name: HackerAI - Architect

    Base Model: gdisney/mistral-nemo-uncensored:latest

    System Prompt: You are a senior security architect. Correlate data points from multiple massive logs to find chained vulnerabilities. Your responses must be highly logical.

    Advanced > num_ctx: 32768

    Advanced > num_gpu: (Leave completely blank. This prevents 16GB RAM systems from crashing out of memory).

Click Save for each. You can now @mention these models in the chat or swap between them depending on if you need strategy, code, or deep log parsing!
🚀 Capabilities & Future Expansion

This base setup gives you a powerful, private, and uncensored multi-model chat interface, but the true power of Open WebUI is its extensibility. Once you are comfortable with this foundation, you can expand it into a fully autonomous agentic workflow:

    Custom Python Filters: You can write middleware scripts that intercept the conversation. For example, you can create a "Context Manager" that automatically hides massive log dumps from the AI's memory window to keep the chat running fast.

    Web Search Integration (CVE Lookups): By linking a free search API (like Tavily or DuckDuckGo) in the Open WebUI settings, your AI can actively search the internet for the latest PoC exploits or query the NVD database in real-time.

    Function Calling (The Agentic Loop): You can write custom Python "Tools" that the AI can trigger on its own. For example, you can build an "Nmap Tool" that allows the AI to literally run port scans on your network, read the results, and automatically suggest an attack path—all from a single prompt in your chat window.

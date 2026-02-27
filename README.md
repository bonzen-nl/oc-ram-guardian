# RAM Guardian — OpenClaw Skill

**Autonoom RAM & memory monitor met intelligente cleanup-triggers.**

Houdt continueel RAM/Swap-gebruik in de gaten, triggert automatische cleanup when thresholds exceed, rapporteert intelligently via Telegram.

---

## 🎯 Wat doet RAM Guardian?

RAM Guardian is een **autonome geheugenmonitor** die:

- **Continuously monitort** — Controleert RAM/Swap elke 5 minuten
- **Intelligente cleanup** — Unload Ollama models, clear caches, optimize ChromaDB
- **Thresholds & Alerts** — Soft cleanup @ 75% RAM, critical alert @ 90%
- **Swap Optimization** — Detecteert swap pressure, analyzes root cause
- **Dutch Reporting** — Mistral-generated analyses in Nederlands
- **Telegram Notifications** — Real-time alerts naar Bob

### 🔄 Monitoring Cycle

```
Every 5 minutes:
    ↓
Check RAM/Swap/Disk
    ↓
Below thresholds? → Sleep 5 min
    ↓
RAM > 75% → Soft Cleanup
    ├─ Unload oldest Ollama model
    ├─ ChromaDB optimization
    ├─ Clear __pycache__
    └─ Generate report
    ↓
RAM > 90% → Critical Alert
    ├─ Recommend: Close Safari, Chrome, Slack
    ├─ System status snapshot
    └─ Alert Bob via Telegram
```

---

## 📦 Afhankelijkheden

### Systeemvereisten
- **Python:** 3.8+
- **Ollama:** http://127.0.0.1:11434 (voor model management)
- **macOS:** system_profiler (for M-chip temperature)

### Python Dependencies

```
psutil>=5.9.0                 # CPU/RAM/Disk metrics
requests>=2.28.0              # Ollama JSON-RPC calls
python-dotenv>=0.20.0         # .env loading
```

### Ollama Requirements
- Models must be running for monitoring
- Guardian unloads models on cleanup (keep_alive: 0)

---

## ⚡ Quickstart

### 1. Installatie

```bash
# Clone repository
git clone https://github.com/bonzen-nl/oc-ram-guardian
cd oc-ram-guardian

# Virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Dependencies
pip install -r requirements.txt
```

### 2. Configuratie

```bash
# Copy template
cp .env.example .env

# Instellingen:
# TELEGRAM_CHAT_ID=your_chat_id
# OLLAMA_BASE_URL=http://127.0.0.1:11434
# RAM_THRESHOLD_PERCENT=75
```

### 3. Setup LaunchAgent (Auto-start macOS)

```bash
# One-time setup
python3 scripts/install_launchagent.py

# Verify
launchctl list | grep ram-guardian

# View logs
tail -f /tmp/ram_guardian.log
```

### 4. Test Monitor

```bash
# Manual check (don't wait 5 min)
python3 scripts/ram_guardian.py --now

# Output: Current RAM/Swap status + metrics
```

---

## 🚀 Gebruik

### Manual Run

```bash
# Check current status
python3 scripts/ram_guardian.py --now

# Force cleanup (even if RAM < 75%)
python3 scripts/ram_guardian.py --force-cleanup

# Verbose logging
python3 scripts/ram_guardian.py --now --verbose
```

### Automatic Monitoring

Once LaunchAgent installed, runs automatically every 5 minutes:

```bash
# View real-time logs
tail -f /tmp/ram_guardian.log

# Check if daemon is running
ps aux | grep ram_guardian
```

### Telegram Alerts

Receives automatic messages like:

```
⚠️ RAM ALERT
RAM: 78% (8.5GB / 10.9GB)
Swap: 45% (3.2GB / 7.1GB)

Action: Unloading mistral-small3.1:24b
Expected: RAM drop to ~45%

📊 System Analysis:
Pressure from sustained model loading.
Recommendation: Close Safari/Chrome to reduce background memory use.
```

---

## 🏗️ Projectstructuur

```
oc-ram-guardian/
├── SKILL.md                          # Skill documentatie
├── README.md                         # Dit bestand
├── requirements.txt                  # Python dependencies
├── .env.example                      # Configuration template
├── .gitignore                        # Git security
├── LICENSE                           # MIT
├── config/
│   └── ram_guardian.json             # Monitoring settings
├── scripts/
│   ├── ram_guardian.py               # Main monitor loop
│   ├── cleanup_engine.py              # Cleanup logic
│   ├── install_launchagent.py        # macOS auto-start
│   └── ollama_manager.py             # Model unloading
├── lib/
│   ├── metrics.py                    # RAM/Disk collection
│   ├── analyzer.py                   # Mistral analysis
│   └── notifier.py                   # Telegram alerts
└── .venv/                            # Virtual environment
```

---

## 📊 Monitoring Metrics

### Collected
- **RAM:** GB + percentage used
- **Swap:** GB + percentage used
- **CPU Load:** 1min, 5min, 15min average
- **Disk:** Free space
- **Temperature:** M-chip (if available)
- **Ollama:** Running models + TPS

### Thresholds

| Metric | Soft Cleanup | Critical Alert |
|--------|---|---|
| **RAM** | 75% | 90% |
| **Swap** | 85% | 90% |
| **Disk** | 90% full | 95% full |

---

## 🧹 Cleanup Sequence

When triggered, RAM Guardian:

1. **Unload Ollama** — Free LLM models (keep_alive: 0)
2. **ChromaDB Optimize** — Dedup vectors, integrity check
3. **Cache Wipe** — 700+ __pycache__ dirs removed
4. **Temp Files** — Clear /tmp/openclaw_* files
5. **Report** — Mistral analyzes & generates summary
6. **Alert** — Telegram notification with results

### Cleanup Results Example

```
✅ Cleanup Complete

Ollama Unloaded:
  • mistral-small3.1:24b (freed 4.2GB)

ChromaDB Optimized:
  • 99 documents scanned
  • 0 duplicates found
  • Database integrity: ✓

Caches Cleared:
  • 712 __pycache__ directories
  • 45 temp files
  • ~800MB freed

Memory Impact:
  • Before: 78% RAM
  • After: 23% RAM
  • Freed: ~7.6GB
```

---

## 🔐 Veiligheid & Permissions

### LaunchAgent Configuration

```bash
# Runs as user (not root)
~/Library/LaunchAgents/nl.openclaw.ram-guardian.plist

# Permissions: 600 (user only)
chmod 600 ~/Library/LaunchAgents/nl.openclaw.ram-guardian.plist
```

### Sensitive Data
- .env (Telegram token) — Never commit
- Logs contain only metrics (no secrets)
- System calls are safe (no destructive writes)

---

## 🧪 Testing

### Unit Tests

```bash
# Test metrics collection
python3 -m pytest tests/test_metrics.py

# Test cleanup logic
python3 -m pytest tests/test_cleanup.py -v

# Test Telegram integration
python3 -m pytest tests/test_notifier.py
```

### Manual Testing

```bash
# Dry-run (show what would happen, don't execute)
python3 scripts/ram_guardian.py --dry-run

# Force cleanup with logging
python3 scripts/ram_guardian.py --force-cleanup --verbose
```

---

## 🐛 Troubleshooting

### LaunchAgent Not Running
```bash
# Check status
launchctl list | grep ram-guardian

# Restart
launchctl stop nl.openclaw.ram-guardian
launchctl start nl.openclaw.ram-guardian

# View errors
log stream --predicate 'process == "ram_guardian"'
```

### Telegram Not Sending Alerts
- Check `.env` has valid TELEGRAM_CHAT_ID
- Verify Telegram bot token in workspace/.env
- Test: `python3 scripts/test_telegram.py`

### Ollama Not Responding
```bash
# Check if Ollama daemon running
ollama ps

# Start if needed
nohup ollama serve > /tmp/ollama.log 2>&1 &
```

### High RAM Still After Cleanup
- Check for runaway processes: `ps aux | sort -k3 -r | head`
- Safari/Chrome hogging memory? Guardian recommends closing
- Need manual intervention if > 95%

---

## 🔗 Sub-Projecten & Integraties

RAM Guardian is onderdeel van het **OpenClaw Skills Ecosystem**:

### Master Hub
- **[oc-overzicht](https://github.com/bonzen-nl/oc-overzicht)** — Central index

### Gerelateerde Skills
- **[oc-software-architect](https://github.com/bonzen-nl/oc-software-architect)** — Uses RAM metrics for task routing
- **[oc-server-status](https://github.com/bonzen-nl/oc-server-status)** — Shares metrics
- **[oc-openclaw-expert](https://github.com/bonzen-nl/oc-openclaw-expert)** — Target for cleanup
- **[oc-github-manager](https://github.com/bonzen-nl/oc-github-manager)** — Can log alerts as GitHub issues

### Integration Points

**Software-Architect checks RAM before heavy tasks:**
```python
ram_available = architect.check_system_memory()
if ram_available < 50%:
    task.defer()  # Wait for cleanup
```

**GitHub Manager logs critical RAM events:**
```python
if ram_pressure == "critical":
    github_mgr.create_issue(
        title="⚠️ Critical RAM pressure event",
        description="System exceeded 90% RAM usage"
    )
```

---

## 📈 Performance Impact

- **Monitoring overhead:** <5% CPU (5 min intervals)
- **Cleanup duration:** 30-60 seconds
- **Memory freed:** 50-80% (per cleanup)
- **User impact:** Minimal (runs in background)

---

## 📝 Licentie

MIT © 2026 Bonzen

---

## 📬 Ondersteuning

- **Issues:** [oc-ram-guardian/issues](https://github.com/bonzen-nl/oc-ram-guardian/issues)
- **System Help:** Zie [oc-server-status](https://github.com/bonzen-nl/oc-server-status)

---

**Onderdeel van:** [OpenClaw Skills Suite](https://github.com/bonzen-nl/oc-overzicht)

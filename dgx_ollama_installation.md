This documentation provides a step-by-step technical guide to deploying **Ollama** on the **NVIDIA DGX Spark**, installing the **DeepSeek-R1 70B** model, and configuring the environment for remote API access.

---

# DGX Spark: Ollama & DeepSeek-R1 70B Setup Guide

## 1. Prerequisites

* **DGX Spark** running DGX OS 7+ (Ubuntu-based).
* **Grace Blackwell (ARM64)** architecture verification.
* Stable internet connection for model downloads (~42GB for 70B).

## 2. Ollama Installation (ARM64 Optimized)

While the standard script works, we must ensure the ARM64 binaries are correctly mapped for the Grace Blackwell chip.

1. **Run the Official Installer:**
```bash
curl -fsSL https://ollama.com/install.sh | sh

```


2. **Verify GPU Visibility:**
Confirm Ollama can see the Blackwell GPU:
```bash
nvidia-smi
# Then check Ollama logs
journalctl -u ollama --no-pager | grep -i "cuda"

```



---

## 3. Configuring Remote API Access

By default, Ollama only listens on `localhost:11434`. To use it as a server for your agentic workflow, you must modify the system service.

1. **Edit the Systemd Service:**
```bash
sudo systemctl edit ollama.service

```


2. **Insert the Environment Variables:**
In the editor, add the following lines to the `[Service]` block to allow remote connections and optimize for Spark's 128GB unified memory:
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_ORIGINS=*"
Environment="OLLAMA_NUM_PARALLEL=4"

```


3. **Reload and Restart:**
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama

```



---

## 4. Installing DeepSeek-R1 70B

For corporate finance, the 70B model provides the "Distilled Llama" or "Distilled Qwen" reasoning capabilities necessary for auditing.

1. **Pull the Model:**
```bash
ollama pull deepseek-r1:70b

```


2. **Test the Installation:**
Run a quick local test to verify the model loads into the 128GB memory pool:
```bash
ollama run deepseek-r1:70b "Calculate the weighted average cost of capital (WACC) if equity is $1M (cost 10%) and debt is $500k (cost 5%)."

```



---

## 5. API Implementation (Python)

You can now call your DGX Spark from any machine on your network.

### A. Using the Ollama Python Library

```python
import ollama

# If running on the DGX itself
client = ollama.Client(host='http://localhost:11434')

# If calling from a separate laptop
# client = ollama.Client(host='http://<DGX_SPARK_IP>:11434')

response = client.generate(
    model='deepseek-r1:70b',
    prompt='Analyze the attached financial markdown table for Q3 revenue leaks.',
    stream=False
)
print(response['response'])

```

### B. Using Raw cURL (REST API)

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "deepseek-r1:70b",
  "prompt": "What is the standard formula for EBITDA?",
  "stream": false
}'

```

---

## 6. Optimization for Corporate Finance

* **Context Length:** DeepSeek-R1 supports large contexts. If your Excel files are massive, ensure you set `"num_ctx": 32768` in your API request options.
* **Format:** Always request `format: "json"` in the API if you need the agent to pipe data into another spreadsheet automatically.


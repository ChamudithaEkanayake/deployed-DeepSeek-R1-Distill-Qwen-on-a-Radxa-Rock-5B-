# DeepSeek-R1 Distill LLM Deployment on Radxa Rock 5B+

This README provides step-by-step instructions to **deploy and use the DeepSeek-R1 Distill Qwen-1.5B model** on a **Radxa Rock 5B+** board. The guide is organized into **local deployment** and **server deployment** sections, making it easy for beginners to follow.

---

## Prerequisites

Before starting, ensure you have:
- A Radxa Rock 5B+ board with Ubuntu or a compatible OS installed.
- Required dependencies: Python 3, Flask, and the RKNN toolkit.
- Model file: `DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm`.
- Workshop directory: `/home/radxa/Desktop/project_LLM/rkllm_server_workshop`.

---

# 1) Local Deployment

### Main Build and Commands

Use these commands to prepare and run the model locally.

| Step | Command | Description |
|------|---------|-------------|
| 1. Build | `./build_rkllm_server_flask.sh --workshop /home/radxa/Desktop/project_LLM/rkllm_server_workshop --model_path /home/radxa/Desktop/project_LLM/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm --platform rk3588` | Builds the RKLLM server with the specified model and platform. |
| 2. Fix Frequencies | `sudo ./fix_freq_rk3588.sh` | Optimizes CPU/GPU frequencies for better performance. |

---

### Set Environment PATH

To ensure the runtime library is accessible:

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/radxa/Downloads/rknn-llm/examples/rkllm_server_demo/rkllm_server/lib
```

> **Note:** Verify that `librkllmrt.so` exists in `/home/radxa/Downloads/rknn-llm/examples/rkllm_server_demo/rkllm_server/lib/`. If not, re-check your RKNN installation.

---

### Local LLM Execution (Direct C++)

If you want to test the model **without setting up a Flask server**, you can run it directly using the provided C++ demo program.

| Context Size | Command | Description |
|--------------|---------|-------------|
| Large (Recommended) | `./llm_demo DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096` | Runs the model with a larger context size. This allows for longer and better-quality conversations, but uses more memory. |
| Small | `./llm_demo DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 128 512` | Runs with a smaller context size, which is faster and lighter but may give shorter or less accurate answers. |

> **Tip for Beginners:** Think of **context size** as how much “memory” the model has during a conversation. A bigger context means it can remember more, but it also needs more resources.

This mode is great for **quick testing** or debugging without needing a server setup.

---

# 2) Server Deployment

### Main Build and Run Commands

To run the model as a server that accepts requests:

1. Build the server:
   ```bash
   ./build_rkllm_server_flask.sh --workshop /home/radxa/Desktop/project_LLM/rkllm_server_workshop --model_path /home/radxa/Desktop/project_LLM/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm --platform rk3588
   ```

2. Fix frequencies:
   ```bash
   sudo ./fix_freq_rk3588.sh
   ```

3. Run the server:
   ```bash
   python3 flask_server.py \
       --rkllm_model_path /home/radxa/Desktop/project_LLM/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm \
       --target_platform rk3588 \
       --prompt_cache_path /home/radxa/Desktop/project_LLM/rkllm_server_workshop
   ```

---

### Server-Client Setup

1. **Open a new terminal** so the server keeps running in the background.
2. Edit the client script `chat_api_flask.py` and update your server’s IP address:
   ```python
   server_url = "http://192.168.8.109:8080/rkllm_chat"  # Replace with your actual IP
   ```
3. Run the client:
   ```bash
   python3 chat_api_flask.py
   ```

Example output:
```bash
(base) radxa@rock-5b-plus:~/Downloads/rknn-llm/examples/rkllm_server_demo$ python3 chat_api_flask.py
```

✅ Now, your client sends prompts to the server over HTTP and displays the model’s responses.

---

### Server-Client Flow Diagram

```
+--------------------+            HTTP/REST           +--------------------+
|  Chat Client       | <----------------------------> |  RKLLM Server      |
|  (chat_api_flask)  |                              |  (flask_server.py) |
+--------------------+                              +--------------------+
         |                                                 |
         | Input: "Hello"                                  |
         |------------------------------------------------>|
         |                                                 |
         | Response: "Hi! How can I help you?"            |
         |<------------------------------------------------|
         | Display output in client                        |
```

This shows the two-way communication: **client sends prompts**, the **server runs the LLM**, and then sends the **response back**.

---

## Troubleshooting

Common issues and solutions:

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| **Server Fails to Start** | Missing library or incorrect path. | Check `LD_LIBRARY_PATH` and verify `librkllmrt.so` exists. |
| **Model Not Found** | Wrong model path. | Double-check the `--rkllm_model_path` argument. |
| **Client Connection Error** | Wrong IP or server not running. | Use `ifconfig` to confirm your IP and update `chat_api_flask.py`. |
| **High Latency** | CPU/GPU not optimized. | Run `sudo ./fix_freq_rk3588.sh` before starting. |
| **Process Won’t Kill** | Insufficient permissions. | Use `sudo kill -9 <PID>` after finding with `ps aux | grep rknn`. |
| **Out of Memory** | Context size too large. | Try smaller context size (e.g., 128/512). |
| **Flask Import Error** | Flask not installed. | Run `pip install flask`. |

---

## Killing Processes

If the server or demo hangs, stop the RKNN processes manually:

1. Find the process:
   ```bash
   ps aux | grep rknn
   ```

2. Kill it:
   ```bash
   sudo kill -9 <process_ID>
   ```

> ⚠️ Use `kill -9` only if normal termination doesn’t work.

---

This README is designed so that **any beginner can set up, test, and deploy the model** on Radxa Rock 5B+. For the latest updates, refer to the official RKNN documentation.


# DeepSeek-R1 Distill LLM Deployment on Radxa Rock 5B+

This documentation provides step-by-step instructions to **deploy and use the DeepSeek-R1 Distill Qwen-1.5B rkllm model** on a **Radxa Rock 5B+** board. The guide is organized into **Local Deployment** (direct execution on board) and **Server Deployment** (Flask server with client).
<br><br>
*Developed by Chamuditha Ekanayake | GitHub: [ChamudithaEkanayake](https://github.com/ChamudithaEkanayake) | LinkedIn: [Chamuditha Ekanayake](https://www.linkedin.com/in/chamuditha-ekanayake-8b37602b6/)*

---

## Prerequisites

Before starting, ensure you have:

- A Radxa Rock 5B+ board with Ubuntu or a compatible OS installed.
- Required dependencies: Python 3, Flask, and the RKLLM runtime.
- Model file: [`DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm`](https://github.com/ChamudithaEkanayake) (converted for RK3588).
- Workshop directory: `/home/radxa/Desktop/project_LLM/rkllm_server_workshop`.

---

# 1) Local Deployment (Direct Execution on Board)

This mode runs the model directly on the Radxa Rock 5B+ without a server. It is useful for **quick testing, benchmarking, or standalone applications**.

### Folder Structure

```
Desktop/
├── project_LLM/
│   ├── DeepSeek-R1-Distill-Qwen-1.5B/
│   │   ├── deploy/
│   │   │   ├── src/llm_demo.cpp
│   │   │   ├── CMakeLists.txt
│   │   │   ├── build/
│   │   │   ├── build-linux.sh
│   │   └── export/
│   │       └── generate_data_quant.py
│   └── rkllm_server/
│       │   ├── rkllm_server
│       │   │   ├── lib
│       │   │   │   ├── librkllmrt.so
│       │   ├── fix_freq_rk3588.sh
│       │   ├── gradio_server.py
│       │   ├── flask_server.py
│       │   └── build_rkllm_server_flask.sh
│   └── rkllm_server_workshop
```

### Build and Run llm_demo (C++)

Go to deploy folder and create build folder:

```bash
cd project_LLM//DeepSeek-R1-Distill-Qwen-1.5B/deploy
mkdir build
cd build
```

Run CMake and Make:

```bash
cmake ..
make -j$(nproc)
```

<!--

Run LLM inference:

```bash
./llm_demo /full/path/to/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 128 512
```


Arguments:

- `model_path` → RKLLM model file
- `max_new_tokens` → e.g., 128
- `max_context_len` → e.g., 512

⚠️ Error: If you see weight tensor errors, ensure the model matches the RKNN runtime version and target platform.

-->

### Deploy to the Board
<!--
Copy the compiled demo and model files to your Radxa board:

```bash
adb push install/demo_Linux_aarch64 /data
adb push DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm /data/demo_Linux_aarch64
adb push ../../scripts/fix_freq_rk3588.sh /data/demo_Linux_aarch64
```

### Running the Demo on Board

<!--
1. Enter the demo directory:

```bash
cd /data/demo_Linux_aarch64
```
-->

1. Set the library path:

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/radxa/Downloads/rknn-llm/examples/rkllm_server_demo/rkllm_server/lib
```

2. (Optional) Lock CPU/GPU frequencies for stable performance:

```bash
sh fix_freq_rk3588.sh
```

3. Run the demo:

```bash
cd project_LLM//DeepSeek-R1-Distill-Qwen-1.5B/deploy/build
export RKLLM_LOG_LEVEL=1
./llm_demo DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm 2048 4096
```

- `2048` = max new tokens to generate
- `4096` = max context length

Arguments:

- `model_path` → RKLLM model file
- `max_new_tokens` → e.g., 128
- `max_context_len` → e.g., 512

⚠️ Error: If you see weight tensor errors, ensure the model matches the RKNN runtime version and target platform.

### Restoring Defaults After Fixing Frequencies

If you ran `fix_freq_rk3588.sh`, the CPU/GPU frequencies are locked. To restore defaults:

- **Reboot the board:**

```bash
sudo reboot
```

- **Or manually reset CPU governor:**

```bash
echo ondemand | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

> ⚠️ GPU scaling usually resets only after reboot. Reboot is the simplest safe option.

---

# 2) Server Deployment (Flask API)

This mode runs the model as a Flask server so you can interact with it from a client program over HTTP.

### Main Build and Run Commands

1. Build the server:

```bash
cd /home/radxa/Desktop/project_LLM/rkllm_server
./build_rkllm_server_flask.sh --workshop /home/radxa/Desktop/project_LLM/rkllm_server_workshop --model_path /home/radxa/Desktop/project_LLM/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm --platform rk3588
```

2. Optimize CPU/GPU frequencies:

```bash
cd /home/radxa/Desktop/project_LLM/rkllm_server/rkllm_server
sudo ./fix_freq_rk3588.sh
```

3. Run the Flask server:

```bash
cd ..
python3 flask_server.py --rkllm_model_path /home/radxa/Desktop/project_LLM/DeepSeek-R1-Distill-Qwen-1.5B_W8A8_RK3588.rkllm --target_platform rk3588 --prompt_cache_path /home/radxa/Desktop/project_LLM/rkllm_server_workshop
```

### Server-Client Setup

1. **Open a new terminal** so the server keeps running.
2. Edit the client script `chat_api_flask.py` to match your server IP:

```python
server_url = "http://192.168.6.102:8080/rkllm_chat"  # Replace with your actual IP
```

3. Run the client:

```bash
python3 chat_api_flask.py
```

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

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| **Server Fails to Start** | Missing library or path wrong | Check `LD_LIBRARY_PATH` and verify `librkllmrt.so` exists |
| **Model Not Found** | Wrong model path | Double-check `--rkllm_model_path` argument |
| **Client Connection Error** | Wrong IP or server not running | Use `ifconfig` to confirm IP and update `chat_api_flask.py` |
| **High Latency** | CPU/GPU not optimized | Run `sudo ./fix_freq_rk3588.sh` before server |
| **Process Won’t Kill** | Insufficient permissions | Use `sudo kill -9 <PID>` after finding with `ps aux | grep rknn` |
| **Out of Memory** | Context size too large | Try smaller values (128 / 512) |
| **Flask Import Error** | Flask not installed | Run `pip install flask` |

---

## Killing Processes

If the server or demo hangs:

```bash
ps aux | grep rknn
sudo kill -9 <process_ID>
```

> ⚠️ Use `kill -9` only if normal termination doesn’t work.

---

This README is designed so that **any beginner can set up, test, and deploy the model** on Radxa Rock 5B+. For the latest updates, refer to the official RKNN documentation.

---

## References

1. Radxa Documentation – [DeepSeek-R1 Deployment on Rock 5B+](https://docs.radxa.com/en/rock5/rock5b/app-development/rkllm_deepseek_r1)  
2. Hugging Face Model – [DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B)  
3. RKLLM Toolkit Documentation – [Rockchip RKLLM Toolkit](https://github.com/airockchip/rknn-llm)


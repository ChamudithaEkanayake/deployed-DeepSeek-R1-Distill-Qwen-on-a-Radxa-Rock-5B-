# DeepSeek-R1-Distill-Qwen-1.5B Demo
1. This demo demonstrates how to deploy the DeepSeek-R1-Distill-Qwen-1.5B model.
2. The open-source model used in this demo is available at: [DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B)

## 1. Requirements

```
rkllm-toolkit==1.2.0
rkllm-runtime==1.2.0
python >=3.8
```

## 2. Model Conversion

1. Firstly, you need to create `data_quant.json` for quantizing the rkllm model, we use fp16 model generation results as the quantization calibration data.
2. Secondly, you run the following code to generate `data_quant.json`  and export the rkllm model.
3. You can also download the **converted rkllm model**  from [rkllm_model_zoo](https://console.box.lenovo.com/l/l0tXb8), fetch code: rkllm

```bash
cd export
python generate_data_quant.py -m /path/to/DeepSeek-R1-Distill-Qwen-1.5B
python export_rkllm.py
```

## 3. C++ Demo

In the `deploy` directory, we provide example code for board-side inference. 

### 1. Compile and Build

Users can directly compile the example code by running the `deploy/build-linux.sh` or `deploy/build-android.sh` script (replacing the cross-compiler path with the actual path). This will generate an `install/demo_Linux_aarch64` folder in the `deploy` directory, containing the executables `llm_demo`, and the `lib` folder.

```bash
cd deploy
# for linux
./build-linux.sh
# for android
./build-android.sh
# push install dir to device
adb push install/demo_Linux_aarch64 /data
# push model file to device
adb push DeepSeek-R1-Distill-Qwen-1.5B.rkllm /data/demo_Linux_aarch64
# push the appropriate fixed-frequency script to the device
adb push ../../../scripts/fix_freq_rk3588.sh /data/demo_Linux_aarch64
```

### 2. Run Demo

Enter the `/data/demo_Linux_aarch64` directory on the board and run the example using the following code

```bash
adb shell
cd /data/demo_Linux_aarch64
# export lib path
export LD_LIBRARY_PATH=./lib
# Execute the fixed-frequency script
sh fix_freq_rk3588.sh
# Set the logging level for performance analysis
export RKLLM_LOG_LEVEL=1
./llm_demo /path/to/your/rkllm/model 2048 4096

# Running result                                                          
rkllm init start
rkllm init success

**********************可输入以下问题对应序号获取回答/或自定义输入********************

[0] 现有一笼子，里面有鸡和兔子若干只，数一数，共有头14个，腿38条，求鸡和兔子各有多少只？
[1] 有28位小朋友排成一行,从左边开始数第10位是学豆,从右边开始数他是第几位?

*************************************************************************


user:
```


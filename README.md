 [comment]: <>

 <h1 align="center"> NaVILA复现
  </h1>


[comment]: <> (  <h2 align="center">PAPER</h2>)
  <h3 align="center">
  <a href="https://github.com/AnjieCheng/NaVILA">Github-VLA</a>
  | 
  <a href="https://github.com/yang-zj1026/legged-loco">Github-low-level</a>
  |<a href="https://navila-bot.github.io/">Website</a>
  | <a href="https://navila-bot.github.io/static/navila_paper.pdf">Paper</a>
  </h3>
  <div align="center"></div>

<br>

## 安装配置

1. 创建conda环境

```bash
conda create -n navila-eval python=3.10
conda activate navila-eval
```

2. 构建Habitat-Sim & Lab (v0.1.7) 

```bash
# 安装Habitat-Sim 0.1.7，但下面只支持python3.6~3.9，用的3.10需要源码安装
conda install -c aihabitat -c conda-forge habitat-sim=0.1.7 headless

# 安装Habitat-Lab
git clone --branch v0.1.7 git@github.com:facebookresearch/habitat-lab.git
cd habitat-lab
# installs both habitat and habitat_baselines
python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt
python setup.py develop --all
```

### Hugging Face下载模型
* 模型地址[a8cheng/navila-llama3-8b-8f](https://huggingface.co/a8cheng/navila-llama3-8b-8f/tree/main)
* 安装库

```bash
conda activate navila-eval
pip install huggingface_hub
```

* 到[网站](https://huggingface.co/settings/tokens)，获取token
* 运行脚本`python download_huggingface.py`,创建python脚本如下所示

```python
from huggingface_hub import snapshot_download

local_dir = snapshot_download(
    repo_id="a8cheng/navila-llama3-8b-8f",
    local_dir="/home/guanweipeng/NaVILA/navila-llama3-8b-8f",
    cache_dir="/home/guanweipeng/NaVILA/navila-llama3-8b-8f/cache",
    token="hf_******",     # ✅ 在这里传 token
    endpoint="https://hf-mirror.com"   # 如果需要走镜像
)

print("模型下载到本地路径:", local_dir)
```



## 实验测试



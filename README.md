 <h1 align="center"> NaVILA复现
  </h1>


[comment]: <> (  <h2 align="center">PAPER</h2>)
  <h3 align="center">
  <a href="https://github.com/AnjieCheng/NaVILA">Github-VLA</a>
  | 
  <a href="https://github.com/yang-zj1026/legged-loco">Github-low-level</a>
  | 
  <a href="https://github.com/yang-zj1026/NaVILA-Bench">NaVILA-Bench</a>
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
# conda install -c aihabitat -c conda-forge habitat-sim=0.1.7 headless

# 安装Habitat-Lab
git clone --branch v0.1.7 git@github.com:facebookresearch/habitat-lab.git
cd habitat-lab
# installs both habitat and habitat_baselines
python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
# 注意，其中的tensorflow==1.13.1似乎已经不支持安装了，改为tensorflow>=2.8.0
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt
python setup.py develop --all
```


* 对于Habitat-sim的源码安装，参考[Link](https://github.com/facebookresearch/habitat-sim/blob/v0.1.7/BUILD_FROM_SOURCE.md)

```bash
git clone git@github.com:facebookresearch/habitat-sim.git #（默认就是v0.1.7）
cd habitat-sim
# git submodule update --init --recursive
# git checkout v0.1.7
# git submodule update --init --recursive #注意切换分支后可能导致部分submodule无效
conda activate navila-eval
# 为了解决NumPy的问题，运行下面：
python ../evaluation/scripts/habitat_sim_autofix.py # 更改habitat-sim/habitat_sim/utils/common.py  (更新的代码已经更改了)

pip install -r requirements.txt
# 如果出现路径问题编译不成功，可能因为之前编译过了，进入到habitat-sim目录删除build(rm -rf build)

# python setup.py install
# python setup.py install --headless # without an attached display
pip install cmake
# sudo apt-get install -y --no-install-recommends \
#      libjpeg-dev libglm-dev libgl1-mesa-glx libegl1-mesa-dev mesa-utils xorg-dev freeglut3-dev
sudo apt update
sudo apt-get install libgl1-mesa-dev
sudo apt-get install libegl1-mesa-dev
pip install --upgrade pybind11
# rm -rf build
python setup.py install --headless --cmake-args="-DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_CXX_STANDARD=11"
```

* Habitat-Sim的源码编译[参考](https://github.com/facebookresearch/habitat-sim/blob/v0.1.7/BUILD_FROM_SOURCE.md)
* 若遇到`error: ‘uint16_t’ in namespace ‘std’ does not name a type; did you mean ‘wint_t’?`报错，请在对应的文件添加：

```cpp
#include <cstdint> // 确保包含uint16_t定义
```

3. 安装VLN-CE依赖

```bash
pip install -r evaluation/requirements.txt
```

4. 安装VILA依赖


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



# 参考资料
* [habitat-sim](https://github.com/facebookresearch/habitat-sim/tree/v0.1.7#installation)
* [habitat-lab](https://github.com/facebookresearch/habitat-lab)
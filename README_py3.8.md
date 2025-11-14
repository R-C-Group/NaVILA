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

~~~
PS: H200没有图形，因此换为4090再次配置
~~~

1. 创建conda环境

```bash
conda create -n navila-eval1 python=3.8
conda activate navila-eval1
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

# #验证
# conda activate navila-eval
# CUDA_VISIBLE_DEVICES=1 python examples/example.py --scene /data/scene_datasets/habitat-test-scenes/skokloster-castle.glb
```


* 对于Habitat-sim的安装

```bash
conda install -c aihabitat -c conda-forge habitat-sim=0.1.7 headless

```

* Habitat-Sim的源码编译[参考](https://github.com/facebookresearch/habitat-sim/blob/v0.1.7/BUILD_FROM_SOURCE.md)
* 若遇到`error: ‘uint16_t’ in namespace ‘std’ does not name a type; did you mean ‘wint_t’?`报错，请在对应的文件添加：

```cpp
#include <cstdint> // 确保包含uint16_t定义
```

安装后用命令查看是否都是0.1.7: `pip list | grep habitat`

3. 安装VLN-CE依赖

```bash
pip install -r evaluation/requirements.txt
```

4. 安装VILA依赖(注意要回到根目录)

```bash
# Install FlashAttention2
pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.5.8/flash_attn-2.5.8+cu122torch2.3cxx11abiFALSE-cp310-cp310-linux_x86_64.whl

# 如果尝试多次都不行可以试试增加超时时间
# pip --default-timeout=1000 install https://github.com/Dao-AILab/flash-attention/releases/download/v2.5.8/flash_attn-2.5.8+cu122torch2.3cxx11abiFALSE-cp310-cp310-linux_x86_64.whl --retries 10

# Install VILA (assum in root dir)
pip install -e .
pip install -e ".[train]" 
pip install -e ".[eval]"

# Install HF's Transformers
pip install git+https://github.com/huggingface/transformers@v4.37.2
site_pkg_path=$(python -c 'import site; print(site.getsitepackages()[0])')
cp -rv ./llava/train/transformers_replace/* $site_pkg_path/transformers/
cp -rv ./llava/train/deepspeed_replace/* $site_pkg_path/deepspeed/
```

5. 修改VLN-CE的WebDataset版本

```bash
pip install webdataset==0.1.103
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

* 参考[VLN-CE](https://github.com/jacobkrantz/VLN-CE)将R2R 和 RxR 数据下载到`evaluation/data`路径；下载方法请参考：[服务器数据下载](https://kwanwaipang.github.io/File/Blogs/Poster/ubuntu%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%8B%E8%BD%BD%E6%95%B0%E6%8D%AE.html)
* 然后下载Matterport3D (MP3D) ，通过[网站](https://niessner.github.io/Matterport/)获取download_mp.py

```bash
# conda create -n mp3d python=2.7
# conda activate mp3d
# python download_mp.py --task_data habitat -o /home/guanweipeng/NaVILA/evaluation/data/scene_datasets/mp3d/ --id 17DRP5sb8fy
python download_mp.py --task habitat -o /home/guanweipeng/NaVILA/evaluation/data/scene_datasets666/mp3d/ --id 17DRP5sb8fy
# python download_mp.py --task minos -o /home/guanweipeng/NaVILA/evaluation/data/scene_datasets6/mp3d/
# 数据实在太大了，也可以尝试从链接：https://cloud.tsinghua.edu.cn/f/03e0ca1430a344efa72b/?dl=1下载，但似乎是没有用的

# 也尝试了用habitat-sim提供的下载但也不work
# conda activate navila-eval
# cd habitat-sim/
# python -m habitat_sim.utils.datasets_download --uids mp3d_example_scene --data-path /home/guanweipeng/NaVILA/evaluation/data/
```

验证R2R：

```bash
cd evaluation
conda activate navila-eval
# bash scripts/eval/r2r.sh CKPT_PATH NUM_CHUNKS CHUNK_START_IDX "GPU_IDS"
bash scripts/eval/r2r.sh /home/guanweipeng/NaVILA/navila-llama3-8b-8f 1 0 "0"
# bash scripts/eval/r2r.sh /home/guanweipeng/NaVILA/navila-llama3-8b-8f 2 0 "1,2"
```

若报错`ImportError: /home/guanweipeng/anaconda3/envs/navila-eval/bin/../lib/libstdc++.so.6: version GLIBCXX_3.4.32 not found (required by /home/guanweipeng/anaconda3/envs/navila-eval/lib/python3.10/site-packages/_magnum.cpython-310-x86_64-linux-gnu.so)`,先查看环境中的libstdc++.so.6是否包含GLIBCXX_3.4.32

```bash
strings /home/guanweipeng/anaconda3/envs/navila-eval/lib/libstdc++.so.6 | grep GLIBCXX_3.4.
#执行安装
conda install -c conda-forge libstdcxx-ng
```

* 对于无显示器情况,或者GPU不能跑图形的情况`unable to find EGL device for cuDA device 0`：

```bash
#安装 OpenGL 开发库
sudo apt install libgl1-mesa-dev
#安装 GLFW 库
sudo apt install libglfw3-dev
# 安装 GLEW 库
sudo apt install libglew-dev

# 安装虚拟显示
sudo apt install xvfb

# 使用Xvfb运行
xvfb-run -a -s "-screen 0 1024x768x24" bash scripts/eval/r2r.sh /home/guanweipeng/NaVILA/navila-llama3-8b-8f 1 0 "0"

#查看安装的pyopengl版本 
pip install --upgrade PyOpenGL PyOpenGL_accelerate
#检查EGL版本
ldconfig -p | grep EGL
ldconfig -N -v | grep libEGL

# 最终通过重新安装habitat-sim不为headlee版本似乎可以解决~
python setup.py install --cmake-args="-DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_CXX_STANDARD=11"
```

* 对于报错跟数据集相关的，比如找不到mp3d或者Navmesh都需要重新确保mp3d数据的下载


但是最终仍然报错：

~~~
Platform::WindowlessEglApplication::tryCreateContext(): unable to find EGL device for CUDA device 0
WindowlessContext: Unable to create windowless context
~~~



可视化的视频存放在：`./eval_out/CKPT_NAME/VLN-CE-v1/val_unseen/videos`

# 参考资料
* [habitat-sim](https://github.com/facebookresearch/habitat-sim/tree/v0.1.7#installation)
* [habitat-lab](https://github.com/facebookresearch/habitat-lab)
* [MatterPort3D 数据集 多途径下载](https://blog.csdn.net/qq_41204464/article/details/149549133)
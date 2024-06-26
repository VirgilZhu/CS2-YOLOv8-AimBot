+ **运行脚本：打开 CS2 ，直接运行 `main.py`即可。**

+ **本机运行脚本的环境：默认 CS2 窗口尺寸（`CSGO_WIN_WH`）在游戏中设定为 1440 * 900，截屏窗口的尺寸（`INFERENCE_WH`）定义为 768 * 480，具体原因如下：**

  1. 由于不同屏幕的尺寸不同，且 win32gui 获取 CSGO 窗口有偏差，若实时更新 `CSGO_WIN_WH` 会导致锁头不准；
  2. **请提前把 `CSGO_WIN_WH` 修改为当前 CSGO 本体的窗口尺寸；**
  3.  `INFERENCE_WH` 是传入模型和截屏窗口的图片尺寸，目前设置为16 : 10的尺寸，和 `CSGO_WIN_WH` 的比例相同，**若当前屏幕尺寸为 16 : 9，请提前修改`INFERENCE_WH`为 (768, 432)；**
  4. **由于截图、检测目标与鼠标移动耗时，锁头并非实时，本机测试时更改了游戏内鼠标 DPI 为 2.5；**
  5. （**注意：**不同屏幕尺寸锁头精准度不同，因为监控 CSGO 窗口所得图片尺寸并不正好等于 CSGO 窗口尺寸，计算有误差）

+ **支持游戏内热键实时更改阵营（不影响炸弹和尸体的检测）和开关功能：**

  + **F5：CT 阵营，检测 T；**
  + **F6：T 阵营，检测 CT；**
  + **F7：Solo，T 和 CT 均检测；**
  + **F8：开关自动移动鼠标；**
  + **F9：开关自动射击。**

  > 剪枝部分代码和项目目录并未上传 Github，根目录文件 AimBot_prune.py 和 AimBot_train.py 仅作参考，在当前环境下无法运行。

  ** **

+ 目录结构：

```
├─datasets			  # 训练数据集
│  ├─test
│  │  ├─images
│  │  └─labels
│  ├─train
│  │  ├─images
│  │  └─labels
│  └─val
│      ├─images
│      └─labels
├─models               # 预训练模型和训练模型权重文件
├─runs				   # 模型训练记录
│  └─detect
│      ├─v8n_150_epoch
│      │  └─weights
│      ├─v8s_100_epoch
│      │  └─weights
│      └─v8s_180_epoch
│          └─weights
```

+ 数据集：
  + 来源：[demModels - v7 (roboflow.com)](https://universe.roboflow.com/sprite-fanta-gpj4f/demmodels/dataset/7)
  + 训练集：2859 张；验证集：269 张；测试集：152 张
  + classes 说明：`{0: "Bomb", 1: "CT", 2: "CT-Head", 3: "Dead", 4: "T", 5: "T-Head"}`
+ 预训练模型：**YOLOv8s**
  + 根据官网文档，l、x 由于模型参数多，推理较慢，n 速度快但精准度最低、s 适中、m 相对较精准；
  + 经过测试，在本机（GPU 为 3060 Laptop）上 m 的 inference 时间长达 20 - 40 ms；150 轮 epoch 训练 n 的 `v8n_150_epoch.pt` 用来推理的耗时与 180 轮 epoch 训练 s 的 `v8s_180_epoch.pt` 大致相同，均在 13 - 18 ms 左右，其差异相对截屏、鼠标移动的耗时可以忽略不计；
  + 故本项目采用 yolov8s 作为预训练模型进行数据集训练，共训练了 180 轮 epoch，权重文件保存在 `models/v8s_180_epoch.pt`。
+ YOLOv8 CLI 命令：
  + 训练：`$ yolo task=detect mode=train model=./models/yolov8s.pt data=./datasets/data.yaml batch=8 epochs=180 imgsz=640 workers=0 device=0`
  + 验证：`$ yolo task=detect mode=val model=./models/v8s_180_epoch.pt  data=./datasets/data.yaml device=0 plots=True`

# Sharpa Developer Documentation

> 技术文档 · Technical Documentation  
> 版本: v1.0 · 更新日期: 2026-06-02  
> 原文: https://sharpa-robotics.github.io/sharpa-docs/

---

## Product Overview

Sharpa 提供完整的灵巧操作平台，包括灵巧手（Wave）、人形机器人（North）和具身AI模型（CraftNet）。

### Sharpa Wave — 22自由度灵巧手

**核心规格：**

| 参数 | 数值 |
|------|------|
| 自由度 (DOF) | 22（15弯曲/伸展 + 6侧向移动 + 1手掌内旋） |
| 指尖力 | >20N |
| 触觉传感器 | 动态触觉阵列（DTA），0.005N灵敏度 |
| 触觉帧率 | 180 FPS |
| 触觉空间分辨率 | <1mm |
| 重量 | 1200g（单手） |
| 尺寸 | 1:1 人手比例 |
| 通信接口 | ROS2-compatible |
| 可靠性认证 | 100万次握持循环无故障 |

**传动架构：** 22电机直驱/QDD架构，所有关节支持反向驱动。直驱+触觉构成正反馈闭环——直驱的力-电信号直接转换特性使触觉感知获得最大保真度。

### Sharpa North — 全自主人形机器人

**核心规格：**

| 参数 | 数值 |
|------|------|
| 高度 | ~170cm |
| 手部 | 双侧 Sharpa Wave（共44 DOF） |
| 全身自由度 | 75 DOF（NVIDIA GR00T参考设计） |
| 计算平台 | NVIDIA Jetson AGX Thor T5000 |
| 操作能力 | 乒乓球对打、纸风车组装、300+次风车连续组装 |
| 演示 | CES 2026 连续4天×8小时无人值守自主演示 |

### CraftNet — 具身AI模型

**架构特征：**

| 参数 | 说明 |
|------|------|
| 模型类型 | Vision-Tactile-Language-Action (VTLA) |
| 子系统 | 三个异步神经网络，运行在不同频率 |
| System 0 | 反射智能（reflex intelligence），毫秒级触觉-运动闭环 |
| System 1 | 快速反应策略，处理常规操作任务 |
| System 2 | 慢速推理规划，处理复杂多步操作 |
| 输入模态 | 视觉（RGB-D）、触觉（DTA阵列）、语言指令 |
| 输出 | 22-DOF关节力矩/位置指令 |
| 兼容性 | ROS2, NVIDIA Isaac Sim, NVIDIA GR00T |

---

## Getting Started

### 环境要求

- **操作系统**: Ubuntu 22.04 LTS (推荐) / Ubuntu 20.04 LTS
- **ROS2**: Humble Hawksbill (推荐) / Foxy Fitzroy
- **Python**: 3.10+
- **CUDA**: 12.0+ (CraftNet推理)
- **Isaac Sim**: 2024.1+ (仿真训练)

### 安装

```bash
# 1. 克隆仓库
git clone https://github.com/sharpa-robotics/sharpa-sdk.git
cd sharpa-sdk

# 2. 安装依赖
pip install -r requirements.txt

# 3. 安装ROS2包
cd ros2_ws
colcon build --symlink-install
source install/setup.bash

# 4. 验证安装
python -c "import sharpa; print(sharpa.__version__)"
```

### 快速测试

```python
from sharpa import WaveHand
import time

# 连接手部
hand = WaveHand(port="/dev/ttyUSB0", hand="right")
hand.connect()

# 复位到零位
hand.go_home()

# 读取触觉数据
tactile = hand.read_tactile()  # shape: (22, 1000+), 0.005N resolution

# 力控抓取
hand.set_torque_mode()
hand.set_joint_torques({
    "index_flexion": 0.5,   # Nm
    "thumb_flexion": 0.3,
})
time.sleep(2.0)
hand.disconnect()
```

---

## Hardware Interface

### 通信协议

Sharpa Wave 通过 USB-C 连接，内部使用 CAN-FD 总线进行电机通信：

- **物理层**: USB 2.0 Type-C
- **协议层**: CAN-FD @ 5Mbps (内部)
- **数据格式**: Protocol Buffers (protobuf)
- **控制频率**: 1 kHz (力控内环) / 100 Hz (位置外环)

### 电机参数

| 参数 | 数值 |
|------|------|
| 电机类型 | 无框力矩电机 (Frameless BLDC) |
| 减速比 | ≤10:1 (准直驱QDD) |
| 额定扭矩 | 0.5-1.5 Nm (关节) |
| 峰值扭矩 | 2-3 Nm |
| 编码器分辨率 | 19-bit 绝对式磁编码器 |
| 控制模式 | 力控 (Torque) / 位置 (Position) / 阻抗 (Impedance) |

### 触觉传感器 (DTA)

每个指尖集成的动态触觉阵列：

```
DTA 规格:
├── 感知单元: >1000 触觉像素/指尖
├── 灵敏度: 0.005N (5mN)
├── 采样率: 180 Hz (全阵列)
├── 空间分辨率: <1mm
├── 感知维度: 法向力 + 切向力 (3-axis)
├── 温度感知: -20°C ~ 80°C (可选)
└── 数据接口: SPI → ARM Cortex-M7 → USB Bulk
```

### 关节运动范围

| 关节 | 运动范围 | 最大速度 |
|------|---------|---------|
| 拇指弯曲/伸展 | 0° ~ 90° | 360°/s |
| 拇指侧向 | -20° ~ 20° | 240°/s |
| 食指弯曲/伸展 | 0° ~ 100° | 360°/s |
| 食指侧向 | -15° ~ 15° | 240°/s |
| 中指弯曲/伸展 | 0° ~ 100° | 360°/s |
| 无名指弯曲/伸展 | 0° ~ 100° | 360°/s |
| 小指弯曲/伸展 | 0° ~ 100° | 360°/s |
| 手腕内旋 | -45° ~ 45° | 180°/s |

---

## Software API Reference

### Python SDK

#### WaveHand Class

```python
class WaveHand:
    """Sharpa Wave 灵巧手主控类"""

    def __init__(self, port: str, hand: str = "right"):
        """
        初始化手部连接
        Args:
            port: 串口设备路径，如 "/dev/ttyUSB0"
            hand: "left" 或 "right"
        """

    def connect(self) -> bool:
        """建立连接，返回成功状态"""

    def disconnect(self) -> None:
        """断开连接，安全停止所有电机"""

    def go_home(self) -> None:
        """复位到零位姿态"""

    # ---- 控制模式 ----
    def set_torque_mode(self) -> None:
        """切换到力矩控制模式 (1kHz)"""

    def set_position_mode(self) -> None:
        """切换到位置控制模式 (100Hz)"""

    def set_impedance_mode(self, stiffness: dict, damping: dict) -> None:
        """切换到阻抗控制模式"""

    # ---- 关节控制 ----
    def set_joint_torques(self, torques: dict) -> None:
        """
        设置关节力矩
        Args:
            torques: {"joint_name": torque_Nm, ...}
        """

    def set_joint_positions(self, positions: dict) -> None:
        """
        设置关节位置
        Args:
            positions: {"joint_name": angle_deg, ...}
        """

    # ---- 触觉感知 ----
    def read_tactile(self) -> np.ndarray:
        """
        读取全手触觉数据
        Returns:
            shape (22, n_pixels), 单位: Newton
        """

    def read_joint_states(self) -> dict:
        """
        读取关节状态
        Returns:
            {"joint_name": {"position": deg, "velocity": deg/s, "torque": Nm}, ...}
        """

    # ---- 状态 ----
    def get_temperature(self) -> dict:
        """读取各关节电机温度 (°C)"""

    def is_connected(self) -> bool:
        """检查连接状态"""

    def emergency_stop(self) -> None:
        """紧急停止，立即切断电机供电"""
```

### ROS2 Topics

| Topic | 类型 | 频率 | 方向 |
|-------|------|------|------|
| `/wave_hand/joint_states` | `sensor_msgs/JointState` | 100Hz | 发布 |
| `/wave_hand/tactile` | `sharpa_msgs/TactileArray` | 180Hz | 发布 |
| `/wave_hand/torque_cmd` | `sharpa_msgs/JointTorque` | 1000Hz | 订阅 |
| `/wave_hand/position_cmd` | `sharpa_msgs/JointPosition` | 100Hz | 订阅 |
| `/wave_hand/temperature` | `sensor_msgs/Temperature` | 10Hz | 发布 |
| `/wave_hand/status` | `sharpa_msgs/HandStatus` | 50Hz | 发布 |

---

## Simulation (Isaac Sim)

### 设置仿真环境

```bash
# 安装 Isaac Sim 扩展
cd ~/isaac_sim/exts
git clone https://github.com/sharpa-robotics/sharpa_isaac_ext.git

# 启动仿真
~/isaac_sim/python.sh -m sharpa_sim.demo

# 或在 Python 中
import isaacsim
from sharpa_sim import WaveHandSim

sim_hand = WaveHandSim(render=True)
sim_hand.reset()
obs = sim_hand.step(torques)
```

### 域随机化参数

用于 sim-to-real 迁移训练：

```python
domain_randomization = {
    "joint_friction": (0.8, 1.2),        # 摩擦系数随机化
    "joint_damping": (0.9, 1.1),          # 阻尼随机化
    "tactile_noise": (0.0, 0.002),        # 触觉噪声 (N)
    "object_mass": (0.7, 1.3),            # 物体质量随机化
    "object_friction": (0.5, 1.5),        # 物体表面摩擦
    "lighting": "random",                 # 光照随机化
    "camera_pose": ("random", 0.02),      # 相机位姿扰动
}
```

---

## NVIDIA GR00T 集成

Sharpa Wave 被选为 NVIDIA Isaac GR00T 参考人形机器人的灵巧手组件：

- **参考设计**: Unitree H2 Plus (31 DOF) + 双侧 Wave (44 DOF) = 75 DOF
- **计算平台**: Jetson AGX Thor T5000
- **仿真**: Isaac Sim 内置 Wave 数字孪生模型
- **训练**: Isaac GR00T 工作流 (模仿学习 → 域随机化 → 仿真训练 → 零样本部署)

```python
# GR00T 集成示例
from gr00t import RobotEnv
from sharpa import WaveHand

# GR00T 自动管理手部接口
env = RobotEnv(
    robot="unitree_h2_plus",
    hands=["sharpa_wave_right", "sharpa_wave_left"],
    sim=True
)

obs = env.reset()
while True:
    action = policy(obs)  # 策略输出 44-DOF 手部力矩
    obs, reward, done, info = env.step(action)
```

---

## 故障排除

### 常见问题

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| 无法连接手部 | USB权限不足 | `sudo chmod 666 /dev/ttyUSB0` 或添加 udev 规则 |
| 触觉数据异常 | 传感器初始化超时 | 重新上电，等待 5 秒初始化完成 |
| 关节异响 | 电机校准丢失 | 运行 `hand.calibrate()` |
| 力控抖动 | 力控增益过高 | 降低 Kp（默认 0.8→0.4），增加 Kd（默认 0.05→0.1） |
| 电机过热 | 连续高负载 | 检查散热，降低 Duty Cycle，确认环境温度<40°C |

### 诊断命令

```bash
# 检查 USB 设备
lsusb | grep Sharpa

# 查看内核日志
dmesg | grep -i sharpa

# 运行自检
python -m sharpa.diagnostics --full

# 测试通信延迟
python -m sharpa.benchmark --test latency
```

---

## 更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-06 | 初始版本，Wave SDK v1.0, GR00T 集成 |
| v0.9 | 2026-03 | 触觉 API 更新，增加阻抗控制模式 |
| v0.8 | 2026-01 | CES 2026 演示版本，North 机器人首次发布 |

---

> **注意**: 由于网络限制，本文档基于公开的 Sharpa 产品信息编写。  
> 官方文档: https://sharpa-robotics.github.io/sharpa-docs/  
> 本文档以 Markdown 格式维护，可通过 PR 更新。

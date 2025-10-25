

# 3D Human Pose Estimation via Intuitive Physics

Some abbreviations：

| Abbreviations | **Full Name**                                    |
| :------------ | ------------------------------------------------ |
| BoS           | *Base of Support*                                |
| CoM           | *Center of Mass*                                 |
| CoP           | *Center of Pressure*                             |
| IP            | *intuitive-physics*                              |
| SMPL          | *Skinned Multi-Person Linear Model*              |
| SOTA          | *State-of the-art*                               |
| HPS           | *human pose and shape*                           |
| MPJPE         | Mean Per-Joint Position Error                    |
| PVE           | Per-Vertex Error                                 |
| MPJPE         | Mean Per-Joint Position Error                    |
| PVE           | Per-Vertex Error                                 |
| PA-MPJPE      | Procrustes Aligned Mean Per-Joint Position Error |
|               |                                                  |
|               |                                                  |





- 为什么论文中多次强调 differentiable?

​	只有可微分，才能利用梯度下降法进行高效优化。

​	如果不可微分，它就像一个黑箱：你给它一个姿态（输入），它告诉你一个结果（比如“摔倒了”或“站稳了”）。但它**不会告诉你**应该如何微调输入的姿态（比如“把左脚往右移动2厘米，膝盖多弯曲3度”）才能让结果变得更好。你无法计算出“稳定性”关于“身体姿态参数”的梯度。



- what's the difference between optimization and regression？

​	PMAN-R（回归）用于快速估计，并在训练时利用地面平面信息；IPMAN-O（优化）用于更精确的拟合，利用已知地面平面直接应用IP损失。

​	实验结果显示，两者都能改善物理真实性（如减少地面穿透或悬浮），但优化在关节指标上可能略逊（因关注表面而非仅关节），而回归在动态姿态上表现更好。论文强调，IP项（如CoM和CoP）是可微分的，便于集成到两者中。



- 为什么使用的模型不同？

> Note that our regression method (IPMAN-R, Sec. 3.4.1) uses SMPL, while our optimization method (IPMAN-O,Sec.3.4.2) uses SMPL-X [63], to match the models used by the baselines.

PMAN-R 使用较简单的 SMPL 模型，而 IPMAN-O 使用更复杂的 SMPL-X 模型。这种选择并非随意，而是为了与它们各自所改进的**baseline保持一致**，从而确保实验结果的**公平性和可比性**，更好地突出他们提出的隐式物理损失的实际效果。



- 为什么要将整体分成部件？

> We introduce a novel CoM formulation that is fully differentiable and **==considers==** the per-part mass contributions, dubbed as pCoM

将身体分成部件是为了**考虑每个部件的质量贡献 (per-part mass contributions)**，从而计算出更真实的重心。

一方面也简化计算（我认为）



- 训练的模型要训练得到哪些参数？

训练模型需要估计SMPL的姿态（θ）和形状（β）参数，相机参数（s, tc, Rc等），以及推导的物理参数（CoM, CoP, BoS）。这些参数通过优化或回归方法从图像数据中学习，并在IPMAN-R和IPMAN-O中分别通过神经网络和目标函数优化得到。具体的权重和超参数（如λ\lambdaλ值）需参考论文的补充材料（Sup. Mat.）。



- ==论文中的部分模型是悬空的，此时如何根据SMPL模型来计算压力？==

​	如果模型完全悬空，压力本质上为零，方法会通过迭代优化降低悬空高度以增加接触

> encouraging plausible floor contact and overlapping CoP and CoM
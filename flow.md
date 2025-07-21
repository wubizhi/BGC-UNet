graph TD
  %% Level‑0
  D0["0. 数据准备层"]

  %% Level‑1 main box
  subgraph cluster_L1["1. 低→高分辨率物理‑生态映射模型 (HighFish‑SR)"]
    direction TB
    D1a["1.1 特征拼接\nLR‑SST/U/V/SHF_QSW + 时序/ENSO/PDO"]
    D1b["1.2 Patch‑wise 0.1° 标签\n4p2z3f + 观测掩膜权重"]
    D1c["1.3 网络结构\n时空 U‑Net + ConvLSTM + CoordConv"]
    D1d["1.4 物理约束损失\n∇·u≈0，Σ生物量≤能量供给"]
    D1e["1.5 交叉验证 & 不确定度 (MC‑Dropout)"]
    D1a --> D1b --> D1c --> D1d --> D1e
  end

  %% Level‑2
  subgraph cluster_L2["2. 未来情景外推"]
    direction TB
    D2a["2.1 输入：CESM‑FEISTY 1° 物理 + 2022‑2100 SSP××"]
    D2b["2.2 批量推理（GPU/分布式）→ HR 4p2z3f"]
    D2c["2.3 情景组装：低/中/高排放三套 HR 数据立方体"]
    D2d["2.4 Ensemble 统计 → 中位值 + 5‑95% 置信区间"]
    D2a --> D2b --> D2c --> D2d
  end

  %% Level‑3
  subgraph cluster_L3["3. 观测资料订正 & 验证"]
    direction TB
    D3a["3.1 观测集成：船载拖网、声学、生物量站点、VIIRS夜光"]
    D3b["3.2 误差学习 ΔB = g(观测 proxy, 模型输出)\n随机森林/GP + 空时核"]
    D3c["3.3 集合 Kalman 滤波：X_corr = X_model + KΔB"]
    D3d["3.4 Leave‑year‑out 验证：2000‑2021 滚动检验"]
    D3e["3.5 生成后验场 + 不确定度栅格"]
    D3a --> D3b --> D3c --> D3d --> D3e
  end

  %% Level‑4
  subgraph cluster_L4["4. 指标提取与应用"]
    direction TB
    D4a["4.1 HotSpot Index：95%分位生物量聚类+漂移矢量"]
    D4b["4.2 FPI 渔获潜力指数：Σ(生物量×可捕率×价系数)"]
    D4c["4.3 多样性/食物网指标：Shannon, TLshift, RRS"]
    D4d["4.4 年际趋势谱：STL + Breakpoint 分析"]
    D4e["4.5 Web‑GIS Dashboard/API 服务"]
    D4f["4.6 管理建议：禁渔线优化、养护区候选、养殖替代"]
    D4a --> D4b --> D4c --> D4d --> D4e --> D4f
  end

  %% Cross‑layer arrows
  D0 --> cluster_L1
  cluster_L1 -->|"训练权重 *.ckpt"| cluster_L2
  cluster_L2 -->|"原始预测场"| cluster_L3
  cluster_L3 -->|"订正后 HR 生态数据集"| cluster_L4

  %% Style tweaks
  classDef box fill:#f9f9f9,stroke:#333,stroke‑width:1px,rx:6,ry:6;
  class D0,D1a,D1b,D1c,D1d,D1e,D2a,D2b,D2c,D2d,D3a,D3b,D3c,D3d,D3e,D4a,D4b,D4c,D4d,D4e,D4f box;

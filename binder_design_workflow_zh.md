# 蛋白质 binder 设计通用流程（中文版）

本文档把 HAO1 项目中验证过的流程整理成可迁移到其他靶点的模板。所有运行输入、脚本、日志和结果都应放在项目目录内；模型权重、参数数据库和大型预测结果放在外部存储，并在 manifest 中记录路径和版本。

## 一、靶点分析与设计策略制定

本步骤的目的是明确：希望 binder 实现什么功能、应结合靶点的哪个区域，以及如何通过热点约束引导设计。应先完成生物学和结构分析，再准备具体运行输入。

**1．明确预期功能和作用机制**

结合靶点的生物学功能，确定 binder 希望实现的效果，例如抑制酶活、阻断配体结合、干扰蛋白相互作用，或限制功能相关的构象变化。

提出可检验的作用机制假设，说明为什么结合某一区域可能产生预期效果。需要区分“与靶点结合”和“结合后发挥功能”，不能仅凭预测结合位置认定功能已经实现。比如设计抑制剂，活性位点可及，那么选择活性位点设计binder当然是好的，如果不可及，不太允许大分子深入口袋抵达活性位点，那么选择可以考虑选择口袋的入口堵住口袋（当然要确定只有这一个主要的入口）或者针对别构位点进行设计，这个时候就需要格外注意，仅仅结合在期望的部位是不够的，还要考虑结合后是否能引起期待的功能变化

**2．收集并分析靶点的功能与结构证据**

查阅相关文献和结构资料，重点了解：

- 功能位点、催化残基、底物或配体结合区域；
- 底物进入通道、功能相关 loop 及构象变化；
- 已知蛋白相互作用界面和调控区域；
- 已有抑制剂、抗体或 binder 的结合位置及作用机制；
- 可用结构的实验状态、配体和辅因子组成、寡聚状态及结构完整性。

据此选择设计参考结构，并记录选择理由。若不同结构在候选结合区域存在明显差异，应比较其构象，必要时分别开展设计。

**3．确定候选结合区域**

根据预期功能和已有证据，选择一个或多个候选区域，说明：

- 该区域与预期功能的关系；
- binder 可能通过何种空间或相互作用机制产生效果；
- 哪些判断已有实验支持，哪些仍属于设计假设。

**4．评估候选区域的空间可及性与结构不确定性**

在完整蛋白及相关生物学组装背景下，检查 binder 是否能够接近候选区域，重点考虑：

- 表面暴露程度、口袋深度和入口宽度；
- 周围结构域、邻近亚基及其他分子的空间遮挡；
- 配体、辅因子或相关修饰对结合空间的影响；
- 缺失残基、柔性区段和不同构象可能造成的遮挡或冲突；
- 截断 target 是否人为暴露了真实蛋白中难以接近的表面。

对于一些在设计位点周围的缺失，应将该区段对候选结合区域的影响列为待验证问题，考虑是不是补全，但是缺失的很多都是柔性很大的loop，预测什么的都预测不准，这个时候更合理的可能是保留缺失进行设计，但是要保留占位，不要让序列直接变连续了。不能因为结构中没有这段坐标，就将其所在空间视为可自由占据的区域。

**5．选择用于空间引导的热点残基**

在具有功能意义且空间可及的区域内选择热点。热点选择应综合考虑：

- 是否位于希望 binder 靠近的区域；
- 表面暴露情况是否支持接触；
- 周围是否存在缺失区段、遮挡或明显的构象不确定性。

功能关键残基与设计热点不必完全相同。对于埋藏较深、难以直接接触的功能残基，可以考虑选择其附近可及的表面残基作为空间引导点。 **这一步可以理解为从空间(几何)角度来引导binder靠近区域，和相互作用没啥关系，在 RFdiffusion 初始热点选择中，通常优先考虑表面可及的疏水或芳香族残基(3-8个)，而不优先选择以极性或带电基团为主的位点。这是因为热点约束主要引导 binder 骨架靠近指定区域，并不能保证后续序列设计形成几何合适的氢键或盐桥。某些极性或带电位点对配对残基的类型、侧链构象及相互作用几何提出了更严格的要求，因此可能缩小后续序列探索中“能够实现有利结合”的可行解空间。此外，这些基团若被埋入界面，却未形成足以补偿水合损失的相互作用，反而可能不利于结合。相比之下，疏水接触通常对特定原子之间的精确方向要求较低，可通过多种合适的侧链组合实现表面贴合与紧密堆积。因此，可优先选择目标功能区域内的疏水残基作为空间引导点，同时保留利用邻近极性残基形成有利接触的机会，并避免将未获得配对的极性基团不合理地埋藏。这属于提高设计成功率的经验策略，并不意味着极性或带电残基不能作为热点。**

**因此我的策略是：在与预期功能相关的候选结合区域内，优先选择邻近关键极性或带电残基、且表面可及的疏水或芳香族残基作为热点，引导 binder 靠近目标区域。这里的“邻近”指三维空间上的接近。通过利用疏水表面的贴合与侧链堆积，提高形成稳定界面的机会，同时为周围极性或带电残基参与结合保留空间。在后续序列设计和结构评价中，再检查这些残基是否能够与 binder 形成几何合理、能够补偿水合损失的氢键或盐桥，从而进一步改善亲和力或特异性。对于未能形成跨界面配对的极性基团，应尽量保留其溶剂可及性，或确认其仍有其他合适的相互作用伙伴，避免因被埋藏却未得到充分稳定而付出能量代价。这样可以减少对某一特定极性配对的依赖，为后续序列和侧链构象探索保留更多可行选择，同时利用条件合适的极性接触优化界面。最终筛选时，应综合检查疏水堆积、极性接触几何及埋藏未满足极性基团，而不能仅依据氢键或盐桥数量判断结合质量。**

逐一记录热点的原始链名、残基编号、氨基酸类型和选择依据，并初步确认其主链坐标是否完整。**热点用于引导 binder 靠近指定区域，不代表所有热点都必须形成直接接触，也不能以热点邻域命中代替功能验证。**

**6．形成设计方案并确定约束**

将上述分析转化为可执行的设计方案，明确：

- 参考结构、目标链及保留范围（经常可以截短，一般只保留热点残基周围10A的距离）；
- binder 长度范围（太长冗余，且容易产生位置冲突；太短难以提供足够大的相互作用面积）；
- 热点组合及其选择依据；
- 需要避开的区域或保留的空间；
- 缺失区段和构象不确定性的处理方案；
- 后续采用截断 target、完整 target 或多构象分别评价的安排。

本步骤完成后，应形成一份设计说明，清楚记录“预期功能—机制假设—候选区域—热点选择”的依据，再进入输入结构准备与质量检查。

## 二、生成和检查 RFdiffusion 骨架

热点约束的 RFdiffusion-v1 设计应使用 complex checkpoint：

```bash
python scripts/generate_rf_campaign.py \
  --hotspots A142,A145,A177,A178,A179,A207,A208,A209 \
  --binder-length 20-30 \
  --num-designs 200 \
  --design-startnum 0 \
  --chunk-size 32 \
  --run-dir output/rf_campaign
```

本实例当前复现实验参数为，要注意的是使用的权重，以及加噪都设置为0：

- `Complex_base_ckpt.pt`；
- `T=50`；
- deterministic sampling；
- `denoiser.noise_scale_ca=0`；
- `denoiser.noise_scale_frame=0`；
- 设计编号按 chunk 连续分配；
- 不在 RF 骨架阶段执行 FastRelax。

每个最终设计必须同时有同名 PDB 和 TRB。staging 目录只放最终骨架，不放扩散轨迹文件。

骨架阶段的热点距离使用 N、CA、C 重建的虚拟 Cβ，统计每个热点到 binder 的最小虚拟 Cβ–Cβ 距离。通常先用 `≤10 Å` 判断是否处于热点邻域，再检查 clash、口袋位置和整体姿态。这个距离是几何筛选指标，不等同于真实结合能。

## 三、使用 ProteinMPNN 设计序列

MPNN 固定 target 链，只设计 binder 链：

```bash
python scripts/run_mpnn_target_fixed.py \
  --rf-dir output/rf_campaign/rfd1_hotspot/staging \
  --run-dir output/mpnn_campaign \
  --num-seq-per-target 10 \
  --model-name v_48_020 \
  --sampling-temp 0.01 \
  --seed 42
```

当前推荐参数：

- 模型：`v_48_020`；
- 设计链：binder 链 B；
- 固定链：target 链 A；
- `sampling_temp=0.01`；
- `batch_size=1`；
- `seed=42`。

脚本会检查链身份（不要错了，有的时候A链对应靶点，有的时候是B链或者别的什么）、target 序列、binder 长度、氨基酸字符和 FASTA 记录数。只有 manifest 状态为 `complete` 时才进入 AF2。

## 四、准备 AF2 initial guess

legacy AF2 initial-guess 实现要求输入 PDB 的链顺序为：

```text
链 A：binder
链 B：target
```

FASTA 可以保持 `target:binder`，但必须在 manifest 中明确记录 FASTA 和 PDB 的链顺序不同。生成输入：

```bash
python scripts/prepare_full_target_inputs.py \
  --run-dir output/mpnn_campaign \
  --rf-dir output/rf_campaign/rfd1_hotspot/staging \
  --target-source rf \
  --target-reference-pdb files/7R4O_target.pdb \
  --missing-target-region A186-A204
```

如果要评估缺失 loop 的空间兼容性，应另建一个使用完整 target 的目录，不能把截断 target 和完整 target 的结果混在同一张表中。

四张 GPU 运行 initial guess：

```bash
AF2IG_RUN_DIR=$PWD/output/mpnn_campaign \
AF2IG_OUT_DIR=$PWD/output/af2_initial_guess_campaign \
AF2_GPUS=0,1,2,3 AF2_RECYCLE=3 \
bash new_hot/run_af2_initial_guess_4gpu.sh
```

当前设置使用 `model_1_ptm`、`recycle=3`。运行结束后应合并各 worker 的 score 文件，并确认预测 PDB 数量等于输入数量、没有 worker 失败。

## 五、initial guess 评价

建议的第一轮结构筛选条件：

- 界面 PAE `≤15 Å`，优先考虑 `<10 Å`；
- binder pLDDT `≥70`，优先考虑 `≥80`；
- binder aligned RMSD `≤2 Å`；
- target aligned RMSD `≤15 Å`，并检查热点局部 RMSD；
- 严重 clash 数为 0；
- 保留每个热点的虚拟 Cβ–Cβ 距离和原始残基映射。

即使 AF2 已经输出结构，也建议继续报告虚拟 Cβ–Cβ 距离。它直接回答 binder 是否仍在指定热点附近；普通原子接触、氢键候选和界面面积则作为补充指标。

## 六、FastRelax 和 Rosetta 界面评价

只对通过 initial guess 初筛的候选执行 FastRelax：

```bash
/home/liangtian/miniconda3/envs/pdesign/bin/python \
  scripts/run_fastrelax_interface_batch.py \
  --pass-csv output/af2_initial_guess_campaign/initial_guess_pass_for_fastrelax.csv \
  --predictions output/af2_initial_guess_campaign/predictions \
  --output-dir output/fastrelax_interface_campaign \
  --hotspots 142,145,177,178,179,207,208,209 \
  --cycles 3
```

当前协议使用 PyRosetta `ref2015`：

- target 主链固定；
- binder 主链允许小幅局部调整；
- 两条链侧链允许重排；
- 链间刚体 jump 固定；
- 保留起始坐标约束；
- 不改变序列。

同时报告：

- Rosetta `interface dG`；
- `dSASA_int`；
- interface packstat；
- interface residue count；
- relax 后的虚拟 Cβ 距离、接触、氢键候选和 clash。

Rosetta 界面分数只能在相同 target 表示和相同协议下做相对排序，不能直接解释为真实结合自由能。

## 七、可选的盲预测

对 initial guess 表现较好的候选，可以分别使用：

1. 截断 target 的盲 AF2/ColabFold；
2. 补全 loop 的完整 target 盲 AF2/ColabFold。

两组必须使用独立输出目录，并比较：界面 PAE、binder pLDDT、target RMSD、binder RMSD、热点距离、界面接触、氢键和 clash。如果候选只在截断 target 上保持结合，应标记为需要 loop 兼容性验证。
**这一步到不是必须的，很多文献都是不做这一步，做完initial guess和Rosetta的一些评估就直接就进实验了**

## 八、可复现性要求

每次 campaign 至少保存：

- 输入结构和序列的路径或；
- RF checkpoint、contig、热点、长度、T、噪声参数、设计编号和 chunk 参数；
- MPNN 模型、温度、batch size 和 seed；
- AF2 模型、recycle、GPU、环境和链顺序；
- 筛选阈值、完整评分表、失败日志和候选 ID；
- PDB/TRB 配对数量和最终 manifest。

每次修改配置都使用新 run 目录。不能只根据“请求生成数量”判断成功，必须检查实际非空 PDB、同名 TRB、评分行和失败 worker。

## 九、特殊

有的靶点在晶体结构中没有解析坐标。截断 target 适合检查 RF 设计姿态能否被 AF2 initial guess 保持，但不能判断真实 loop 是否会与 binder 冲突。完整 target 预测用于空间兼容性分析，不能替代实验验证。

# 复制给服务器Codex：1个32PE tile的局部Attention非MAC快速估算

```text
请使用本包完成一次最小DC综合，获取每32PE tile局部Attention非MAC的延迟模型(ns)、面积(µm²)、估计能量(pJ)，并输出1024tile并行阵列的局部估算。不再对旧16ctx×4dim完整FP32 Attention顶层综合。

先读README.md并执行python3 verify_upload.py。综合顶层为attention_32pe_tile_local_nonmac_eval：一个32-score局部softmax/PV状态接口，HEAD_DIM=128。现有架构的4个维度tile共享这套局部非MAC，所以只映射完整共享单元一次，将面积/功率/能量除4得到一个32PE tile分摊值，再乘1024得到阵列局部总量。禁止把HEAD_DIM降到32并对部分QK先softmax，禁止把该分摊值称为独立tile完整Attention实测。

只用dc_shell一次，单次compile -map_effort medium，DC自带基本STA、面积、Power Compiler无波形功耗估计。不要PT/PrimePower/VCS，不做功能仿真，不找DW仿真模型、不生成VCD/SAIF，不做多N/seed、空闲窗口、Fmax搜索、反复闭合或全局DDC。仅分析sources.f的两个RTL，不综合QK/PV MAC、物理PE、K/V bank或完整head。不停止或删除其他现有服务器任务和结果。

填synopsys/site.json：SMIC SCC28NHKCP_HSC35P140_RVT_V0p2，RVT/SS/0.9V/125C basic NLDM/NLPM，DW综合库/许可。核对report_units、库面积单位。保持1ns理想时钟和constraints.tcl现有条件：setup/hold uncertainty50/20ps，clock/input slew50ps，输入/输出max/min100/0ps，输出负载5fF，max transition200ps；仅rst_n false path，无FSM假路径/multicycle、CTS/SPEF。

python3 synopsys/run_flow.py
python3 synopsys/collect_candidate.py results/smic28_32pe_tile

已有同源同参数同库同约束网表优先用--stage analyze --output <已有目录>复用，不重新compile。允许按本版DC help/man修正测量脚本并留diff，不改RTL算法或规模。保留链接/无黑盒/单位、QoR/约束摘要及setup/hold各10条路径即可，不输出海量逐项违例。

延迟按1ns/周期模型：从已完成128维QK归约的32个score接受后，max5拍、diff1拍、exp1拍，7ns产生beta；PV请求立即接受占1拍，分母5拍与PV重叠，PV及时返回时最少13ns产生局部(m,l,z)。更慢PV/供数/背压等待另计。不能把算术并行度当成1024次串行工作：阵列并行局部延迟仍7/13ns，不乘1024也不除4。另列实际DC时序，即使失败也不改模型值，不宣称闭合1GHz。完整head在N1152时仍沿用121ns命令、79ns非MAC余项模型，本小样本未重新测完整head。

功耗仅在DC/Power Compiler中用set_switching_activity + report_power。非clk/reset输入0.1次翻转/周期、P(1)=0.5，clk每周期2次翻转，rst_n恒1。核对-period语法、活动单位以及动态/漏电报告单位，逐项统一成W。功耗许可或命令不可用时，面积/时序仍交付，功耗及能量null/PENDING并说明，不改用PT或重跑综合。
默认N1152能量统一按E_pJ=P_W×121ns×1000，动态、漏电和总值分列；这是完整head模型窗口内该局部电路的假设能量，不含外部MAC和共享硬件自身功耗。每tile能量=样本/4，1024tile局部总能量=每tile×1024。没有真实命令/FSM/尾wave活动，标VECTORLESS_ASSUMED_ACTIVITY，不标MEASURED。

跨16context行归约、running merge、query缩放、最终倒数/输出缩放、head级控制/状态、全局互连/转换、GSRAM/PHY均未测。完整16head非MAC面积应为256×样本局部面积+16×每head共享面积S，本轮S保持缺失，不当零，不把局部结果当完整PE阵列总面积/Attention总能耗；原MAC成本另计。

交付RESULTS_SUMMARY.md、tile_nonmac.raw_candidate.json，列出样本、每32PE tile分摊及1024tile局部合计的面积/功率/能量，局部延迟模型与真实DC时序状态分列；附关键面积/单位/时序/功耗报告、源码/网表hash和脚本diff。功能仿真SKIPPED_PER_USER_REQUEST；不改模拟器、Excel、RTL或旧数据。得到上述数据即交付，不追加任务。
```

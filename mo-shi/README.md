# 模式

## Hadoop三種模式:

1. 單機模式\(Standalone Mode\):
   * 這種模式下不會啟動任何背景Java程式，此模式適合開發測試及除錯。
2. 偽分佈式模式\(Pseudo-Distributed Mode\)
   * Hadoop中的背景Java程式均運行於本機節點，可以模擬小規模的叢集，[架設步驟請參閱](https://hackmd.io/@JeffWen/bdsevm)。
3. 完全分佈式模式\(Fully-Distributed Mode\)
   * Hadoop中的背景Java程式運行數個主機上。

| 屬性 | 本機模式 | 偽分佈式模式 | 完全分佈式模式 |
| :--- | :--- | :--- | :--- |
| fs.defaultFS | file:/// | hdfs:/// | hdfs:/// |
| dfs.replication | N/A | 1 | 3 |
| mapreduce .framework.name | N/A | yarn | yarn |
| yarn.resourcemanager.hostname | N/A | localhost | resourcemanager |
| yarn.nodemanager.auxervices | N/A | mapreduce\_shuffle | mapreduce\_shuffle |


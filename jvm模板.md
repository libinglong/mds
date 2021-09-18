### jvm模板
* CMS:
  
  * -XX:+UseConcMarkSweepGC
  
  * -XX:+UseParNewGC
  
  * -Xmx8G 最大堆
  
  * -Xms8G 初始化堆
  
  * -Xmn4G 年轻代
  
  * -XX:SurvivorRatio=
  
    eden与survivor比值，注意有2个survivor
  
  * -XX:NewRatio=6
    
  * -XX:TargetSurvivorRatio=80
    Survivor，从年龄为1到n的对象如果超过了80%，那么年龄阈值动态调整为n
  
* G1:

  * -XX:+UseG1GC

  * -XX:G1HeapRegionSize=

  * -XX:+UnlockExperimentalVMOptions

  * -XX:G1NewSizePercent=

  * -XX:SurvivorRatio=

    eden与survivor比值，注意有1个survivor

* Common:
  
  * -Xmx
  * -Xms
  * -server
  * -XX:MaxTenuringThreshold=

  ---

* gc:

  以下参数建议线上全部添加

  * -Xloggc:filename.yyyy.MM.dd.HH.mm.ss.SSS.log

  * -XX:+PrintGC

  * -XX:+PrintReferenceGC

  * -XX:+PrintHeapAtGC

  * -XX:+PrintGCDetails

  * -XX:+PrintGCTimeStamps

    打印时间戳，可以方便的排查safepoint相关问题，因为safepoint日志仅打印时间戳。

  * -XX:+PrintGCDateStamps

  * -XX:+PrintAdaptiveSizePolicy

  * -XX:+PrintGCApplicationStoppedTime

  * -XX:+PrintGCApplicationConcurrentTime

  * -XX:+PrintTenuringDistribution

  ---

* safepoint:

  以下参数建议线上全部添加，Safepoint相关日志会输出至stdout

  * -XX:+SafepointTimeout
  * -XX:SafepointTimeoutDelay=1000
  * -XX:+PrintSafepointStatistics
  * -XX:PrintSafepointStatisticsCount=1

* 偏向锁优化
  
* -XX:-UseBiasedLocking
  
* 其他

  有的时候可能Safepoint相关日志没有输出至stdout，添加下述参数

  * -XX:+UnlockDiagnosticVMOptions
  * -XX:+LogVMOutput
  * -XX:LogFile=vm.log

### 注意事项

* stdout和stderr记得重定向到某文件，加时间戳以免覆盖
* gc日志加时间戳以免覆盖


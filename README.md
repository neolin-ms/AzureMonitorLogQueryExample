# Azure Monitor Log Query Example

## References
https://www.cloudsma.com/2018/07/cpu-processes-azure-log-analytics/<br>
https://docs.microsoft.com/en-us/azure/azure-monitor/logs/examples#performance<br>
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/<br>

## How-To
1.x for Windows<br> 
2.x for Linux<br>

## 1.1 Get a CPU count for your machines
```bash
Perf
| where TimeGenerated >= ago(1h)
| where ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName!="_Total"
| sort by InstanceName asc nulls first
| summarize dcount(InstanceName) by Computer
```

## 1.2 Find High CPU of Virtual Machines
```bash
Perf
| where CounterName == "% Processor Time"
              and CounterValue > 90
              and ObjectName == "Processor"
              and InstanceName == "_Total"
| distinct Computer
```

## 1.3 Check the Processor time of Virtual Machines
```bash
Perf
| where TimeGenerated > now(-60m)
              and ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName != "_Total"
              and InstanceName != "Idle"
              and CounterValue > 50
```
## 1.4 Find instances of total cpu being used above 70% over the last 20 minutes
```bash
//defining our CPU threshold
let CPUThreshold = 70;
//define time sample rate
let Time = 20m;
//define Count of processes to return
let Count = 5;
//Find instances of total cpu being used above 90% over the last 10 minutes
Perf
| where TimeGenerated > now(-Time)
              and ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName == "_Total"
              and CounterValue > CPUThreshold
| project Computer, ObjectName
              , CounterName, CounterValue
              , TimeGenerated;
```
## 1.5 Find top Processes, excluding _Total and Idle instances, there may be othe
```bash
//defining our CPU threshold
let CPUThreshold = 70;
//define time sample rate
let Time = 20m;
//define Count of processes to return
let Count = 5;
// Find top Processes, excluding _Total and Idle instances, there may be other instances you want to exclude as well
Perf
| where TimeGenerated > now(-Time)
               and CounterName == "% Processor Time"
               and InstanceName != "_Total"
               and InstanceName != "Idle"
| project Computer, ObjectName
              , CounterName, InstanceName
              , CounterValue, TimeGenerated;
```

## 1.6 Find CPU count for servers(s)
```bash
//defining our CPU threshold
let CPUThreshold = 70;
//define time sample rate
let Time = 20m;
//define Count of processes to return
let Count = 5;		  
// Find CPU count for servers(s)
Perf
| where TimeGenerated >= ago(1h)
| where ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName!="_Total"
| sort by InstanceName asc nulls first
| summarize CPUCount = dcount(InstanceName) by Computer;
```

## 1.7 Join all 3 datasets together
```bash
FindCPU | join(TopCPU) on Computer 
| join(TopProcess) on Computer
| extend PercentProcessorUsed = CounterValue1 // CPUCount
| summarize avg(PercentProcessorUsed) by Computer, ObjectName
                  , CounterName, CPUCount 
                  , TotalCPU=CounterValue //rename CounterValue to TotalCPU 
                  , Process=ObjectName1 //rename ObjectName1 to Process 
                  , ProcessTime=CounterName1 //rename CounterName1 to ProcessTime 
                  , ProcessName=InstanceName //rename InstanceName to ProcessName 
                  , TimeGenerated
| where Process == "Process"
and avg_PercentProcessorUsed > 25 // only return processes that are using more than 25%
| top Count by avg_PercentProcessorUsed desc
| project Computer, CPUCount
                , ProcessName , avg_PercentProcessorUsed
                , TotalCPU, Process
                , ProcessTime, TimeGenerated
```

## 1.8 Find High CPU Processes in Azure Log Analytics for Windows VM
```bash
let CPUThreshold = 70;
let Time = 20m;
let Count = 5;
let TopCPU = Perf
| where TimeGenerated > now(-Time)
              and ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName == "_Total"
              and CounterValue > CPUThreshold
| project Computer, ObjectName
              , CounterName, CounterValue
              , TimeGenerated;
let TopProcess = Perf
| where TimeGenerated > now(-Time)
               and CounterName == "% Processor Time"
               and InstanceName != "_Total"
               and InstanceName != "Idle"
| project Computer, ObjectName
              , CounterName, InstanceName
              , CounterValue, TimeGenerated;
let FindCPU = Perf
| where TimeGenerated >= ago(1h)
| where ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName!="_Total"
| sort by InstanceName asc nulls first
| summarize CPUCount = dcount(InstanceName) by Computer;
FindCPU | join(TopCPU) on Computer 
| join(TopProcess) on Computer
| extend PercentProcessorUsed = CounterValue1
| summarize avg(PercentProcessorUsed) by Computer, ObjectName
                  , CounterName, CPUCount 
                  , TotalCPU=CounterValue
                  , Process=ObjectName1
                  , ProcessTime=CounterName1
                  , ProcessName=InstanceName
                  , TimeGenerated
| where Process == "Process"
and avg_PercentProcessorUsed > 25
| top Count by avg_PercentProcessorUsed desc
| project Computer, CPUCount
                , ProcessName , avg_PercentProcessorUsed
                , TotalCPU, Process
                , ProcessTime, TimeGenerated
```

## 2.1 Find High CPU on Process of VM 
```bash
Perf
| where Computer == "mycentos77vm0928"
        and ObjectName == "Process"
        and CounterName == "Pct User Time"
        and InstanceName == "stress"
//| where TimeGenerated == todatetime('2021-09-29T04:02:07.283Z')
| sort by TimeGenerated desc  
```

## 3.1 Find the High CPUT of Process by Time range (ago)
```bash
Perf
| where Computer == "myCentOS77vm1004"
    and ObjectName == "Process"
    and CounterName == "Pct User Time"
    and InstanceName == "stress"
    and CounterValue != "0"
| where TimeGenerated > ago(1h)       
| sort by TimeGenerated desc
```

## 3.2 Find the High CPUT of Process by Time range (between)
```bash
Perf
| where Computer == "myCentOS77vm1004"
    and ObjectName == "Process"
    and CounterName == "Pct User Time"
    and InstanceName == "stress"
    and CounterValue != "0"
| where TimeGenerated between (datetime("2021-10-04T14:30:00") .. datetime("2021-10-04T15:30:00")) 
```

## 4.1 Get the same number of records for each category
```bash
Perf
| Where TimeGenerated between (datetime(10/3/2021) .. datetime(10/5/2021))
| where ObjectName in ("Processor Information", "Memory", "LogicalDisk")
| partition by ObjectName (top 10 by Computer)
| summarize count() by ObjectName
```

## 5.1 Find the scale-down events of cluster-autoscaler
### Azure Portal > Kernetes service > Monitor > Inisghts > Diagnostic settings > cluster-autoscaler
```bash
AzureDiagnostics
| where Category == "cluster-autoscaler"
| where log_s !has "returning in-memory size"
| where log_s !has "No candidates for scale down"
| where log_s has "scale_down"
| project TimeGenerated, log_s
| sort by TimeGenerated desc
```

## 6.1 Logging to Azure from an AKS Cluster
### https://trstringer.com/native-azure-logging-aks/
```bash
ContainerLog
| join kind = inner (KubePodInventory
    | project ContainerID, PodName=Name, ControllerKind, ControllerName, Namespace
    | distinct *
) on ContainerID
| project TimeGenerated, Namespace, PodName, LogEntry, LogEntry, LogEntrySource, ControllerKind, ControllerName
| sort by TimeGenerated desc
```

## 7.1 Find the CPU/Memory utilization of Top 10 on Perf of Container Insight
###// 20230411 - cpuUsagePercent - part 1
```bash
let startTimestamp = ago(5m);
Perf
| where TimeGenerated > startTimestamp
| where ObjectName == "K8SContainer"
| as T
| where CounterName == "cpuUsageNanoCores"
| summarize avgUsage=avg(CounterValue) by InstanceName
| extend PodUid = tostring(split(InstanceName, '/')[9]), ContainerName = tostring(split(InstanceName, '/')[10])
| project PodUid, ContainerName, avgUsage, InstanceName
| join
(
  T
  | where CounterName == "cpuLimitNanoCores"
  | summarize avgUsage=avg(CounterValue) by InstanceName
) on InstanceName
| extend cpuUsagePercent = ( avgUsage / avgUsage1 ) * 100
| join kind=leftouter KubePodInventory on PodUid
| summarize by Name, ContainerName, round(cpuUsagePercent, 2)
| top 10 by cpuUsagePercent desc  
```

###//20240411 memoryUsagePercent - part 1
```bash
let startTimestamp = ago(5m);
Perf
| where TimeGenerated > startTimestamp
| where ObjectName == "K8SContainer"
| as T
| where CounterName == "memoryWorkingSetBytes"
| summarize avgUsage=avg(CounterValue) by InstanceName
| extend PodUid = tostring(split(InstanceName, '/')[9]), ContainerName = tostring(split(InstanceName, '/')[10])
| project PodUid, ContainerName, avgUsage, InstanceName
| join
(
  T
  | where CounterName == "memoryLimitBytes"
  | summarize avgUsage=avg(CounterValue) by InstanceName
) on InstanceName
| extend memoryUsagePercent = ( avgUsage / avgUsage1 ) * 100
| join kind=leftouter KubePodInventory on PodUid
| summarize by Name, ContainerName, round(memoryUsagePercent, 2)
| top 10 by memoryUsagePercent desc 
```

###//20230411 memoryUsagePercent - part 2
```bash
Perf
| where ObjectName == "K8SContainer"
| as T
| where CounterName == "memoryWorkingSetBytes"
| summarize avgUsage=avg(CounterValue) by InstanceName, _ResourceId
| extend PodUid = tostring(split(InstanceName, '/')[9]), ContainerName = tostring(split(InstanceName, '/')[10]), ClusterName = tostring(split(_ResourceId, '/')[8])
| project PodUid, ContainerName, ClusterName, avgUsage, InstanceName
| join
(
  T
  | where CounterName == "memoryLimitBytes"
  | summarize avgUsage=avg(CounterValue) by InstanceName
) on InstanceName
| extend memoryUsagePercent = ( avgUsage / avgUsage1 ) * 100
| join kind=leftouter KubePodInventory on PodUid
| summarize by Name, ContainerName, ClusterName, round(memoryUsagePercent, 2)
| top 10 by memoryUsagePercent desc
```

###//20230411 cpuUsagePercent - part 2
```bash
Perf
| where ObjectName == "K8SContainer"
| as T
| where CounterName == "cpuUsageNanoCores"
| summarize avgUsage=avg(CounterValue) by InstanceName, _ResourceId
| extend PodUid = tostring(split(InstanceName, '/')[9]), ContainerName = tostring(split(InstanceName, '/')[10]), ClusterName = tostring(split(_ResourceId, '/')[8])
| project PodUid, ContainerName, ClusterName, avgUsage, InstanceName
| join
(
  T
  | where CounterName == "cpuLimitNanoCores"
  | summarize avgUsage=avg(CounterValue) by InstanceName
) on InstanceName
| extend cpuUsagePercent = ( avgUsage / avgUsage1 ) * 100
| join kind=leftouter KubePodInventory on PodUid
| summarize by Name, ContainerName, ClusterName, round(cpuUsagePercent, 2)
| top 10 by cpuUsagePercent desc 
```

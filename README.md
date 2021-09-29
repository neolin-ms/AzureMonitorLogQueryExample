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

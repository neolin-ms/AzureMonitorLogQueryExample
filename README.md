# Azure Log Analytics

## Get Higih CPU
```bash
Perf
| where TimeGenerated >= ago(1h)
| where ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName!="_Total"
| sort by InstanceName asc nulls first
| summarize dcount(InstanceName) by Computer
```

## Finding CPU Processes and High CPU
```bash
Perf
| where CounterName == "% Processor Time"
              and CounterValue > 90
              and ObjectName == "Processor"
              and InstanceName == "_Total"
| distinct Computer
```
## Check the Process
```bash
Perf
| where TimeGenerated > now(-60m)
              and ObjectName == "Processor"
              and CounterName == "% Processor Time"
              and InstanceName != "_Total"
              and InstanceName != "Idle"
              and CounterValue > 50
```

## Dedicated time slot 
```bash
Perf
| where TimeGenerated == "9/28/2021, 5:32:46.003 PM"
        and Computer == "myRedHat82VM0929"
        and ObjectName == "Process"
        and CounterName != "Pct Privileged Time"
        and InstanceName != "_Total"
```

## Dedicated Computer/Virtual Machine and check the process stress
```bash
Perf
| where Computer == "mycentos77vm0928"
        and ObjectName == "Process"
        and InstanceName == "stress"
```

## Dedicated Computer/Virtual Machine and Check the Process
```bash
Perf
| where Computer == "mycentos77vm0928"
        and ObjectName == "Process"
        and CounterName == "Pct User Time"
        and InstanceName == "stress"
| where TimeGenerated == todatetime('2021-09-29T04:02:07.283Z')
| sort by TimeGenerated desc  	
```
## Find High CPU Processes in Azure Log Analytics for Windows VM
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

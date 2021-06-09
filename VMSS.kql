///////////////// All CRP related queries  //////////////////////////////
/////////////////////////////////////////////////////////////////////////

// cluster('azcrp').database('crp_allprod')

// Use ' | where operationName == "VMScaleSetVMs.VMScaleSetVMsOperation.PUT" ' for CREATE only

// Use this for dashboard: ' | where TIMESTAMP between (_startTime .. _endTime) '

// Condition to determine AKS VMSS: | where resourceName startswith 'aks'

// Use ApiQosEvent_nonGet instead of ApiQosEvent if only getting PUT/POST operations

// To filter CAS and cloud provider calls: userAgent contains 'cluster-autoscaler' or userAgent contains 'kubernetes-cloudprovider'

/////////////////////////////////////
// All VMSS VM Creation CSE errors
/////////////////////////////////////
// Errors are defined in: https://github.com/Azure/AgentBaker/blob/master/parts/linux/cloud-init/artifacts/cse_helpers.sh
// ERR_OUTBOUND_CONN_FAIL=50 {{/* Unable to establish outbound connection */}}
// ERR_K8S_API_SERVER_CONN_FAIL=51 {{/* Unable to establish connection to k8s api server*/}}
// ERR_K8S_API_SERVER_DNS_LOOKUP_FAIL=52 {{/* Unable to resolve k8s api server name */}}
// ERR_K8S_API_SERVER_AZURE_DNS_LOOKUP_FAIL=53 {{/* Unable to resolve k8s api server name due to Azure DNS issue */}}
// ERR_GPU_DRIVERS_START_FAIL=84 {{/* nvidia-modprobe could not be started by systemctl */}}
// ERR_GPU_DRIVERS_INSTALL_TIMEOUT=85 {{/* Timeout waiting for GPU drivers install */}}
// ERR_GPU_DEVICE_PLUGIN_START_FAIL=86 {{/* nvidia device plugin could not be started by systemctl */}}
// ERR_GPU_INFO_ROM_CORRUPTED=87 {{/* info ROM corrupted error when executing nvidia-smi */}}
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
//| where operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| where resourceName startswith 'aks'  // there is no better way to identify AKS vmss other than this for now
//| where resourceGroupName startswith 'mc_'
| where errorDetails contains "'vmssCSE'" and errorDetails contains "status="
| extend status = extract("status=([0-9.]+)", 1, errorDetails)
| summarize count() by status, bin(PreciseTimeStamp, 1d)
| render timechart 

////////////////////////////////////
// VMStartTimeOut
///////////////////////////////////
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where resultCode == 'VMStartTimedOut'
| summarize count() by operationName, bin(PreciseTimeStamp, 1d)  // userAgent 
| render stackedareachart   

[query](https://dataexplorer.azure.com/clusters/azcrp/databases/crp_allprod?query=H4sIAAAAAAAAA12OuwrCQBBFe79iSJWAhdpZRBAVq/ggIa2sm8EsZB/MzhoifrwbiyB2l8u5j7rYOnW1/vBEw7M39C0SwoVQKo+V0liy0A42IB42XS+abGIIfei4GhxCnsPq17eBJB7JBncSGsGzIPa94haSYndLJhSJLO2Rheo8SGuiMB6SuijHxLjenAOPvA9aC1IvjFgwnGZwH+CuTPr/dA7L70dC0yABR1+2sewDtf5hPugAAAA=)

VMApiQosEvent
| where PreciseTimeStamp > ago(90d)
| where resultType == 2
| where resourceGroupName startswith "MC_"
| where errorDetails contains "VMStartTimedOut"
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 



// Ratio
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where operationName in ('VMScaleSetVMs.VMScaleSetVMsOperation.PUT', 'VirtualMachineScaleSets.ResourceOperation.PUT', 'VirtualMachineScaleSets.ResourceOperation.PATCH')
| summarize total=count(), timedOut=countif(resultCode == 'VMStartTimedOut') by bin(PreciseTimeStamp, 1d)  // userAgent 
| extend r = timedOut * 100. / total | project-away total, timedOut
| render columnchart 


cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where resultCode == 'VMStartTimedOut'
| extend r = parse_json(requestEntity) | extend sku=tostring(r.sku.name), size=tostring(r.properties.hardwareProfile.vmSize)
| extend vmSize = iff(isempty(sku), size, sku)
//| where clientApplicationId == 'AzureContainerService'
| project operationName, vmSize, r, labels, apiVersion, userAgent, region, errorDetails, exceptionType, resultCode, resultType, 
    resourceName, resourceGroupName, operationId, correlationId, clientRequestId, PreciseTimeStamp
| summarize count() by vmSize, bin(PreciseTimeStamp, 1d)
| render stackedareachart   

// From non portal (RP or cloud provider or CAS)
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where resultCode == 'VMStartTimedOut'
| extend r = parse_json(requestEntity) | extend size=tostring(r.sku.name)
| where clientApplicationId != 'IbizaPortal'  // Exclude Portal
| project size, r, labels, clientApplicationId, apiVersion, userAgent, region, errorDetails, exceptionType, resultCode, resultType, 
    resourceName, resourceGroupName, operationName, operationId, correlationId, clientRequestId, PreciseTimeStamp
| summarize count() by operationName, bin(PreciseTimeStamp, 1d)

///////////////////////////////////


/////////////////////////////////////
// All non-GET non user errors
////////////////////////////////////
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where resultType == 2
| summarize count() by resultCode, bin(PreciseTimeStamp, 1d)
| render timechart  

/////////////////////////////////////
// All non-GET user errors
////////////////////////////////////
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where resultType == 1
| summarize count() by resultCode, bin(PreciseTimeStamp, 1d)
| render timechart  

/////////////////////////////////////
// All non-GET InternalExecutionError errors, seen spike when outage
////////////////////////////////////
cluster('azcrp.kusto.windows.net').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where resultCode == 'InternalExecutionError'
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 

cluster('azcrp.kusto.windows.net').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where resultCode == 'InternalExecutionError'
| summarize dcount(subscriptionId) by region, bin(PreciseTimeStamp, 1d)
| render timechart 

cluster('azcrp.kusto.windows.net').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where resultCode == 'InternalExecutionError'
| summarize dcount(subscriptionId) by exceptionType, bin(PreciseTimeStamp, 1d)
| render timechart 



//////////////////////////////////////////////////////////////////////////
//Force Delete latency @CRP level
//////////////////////////////////////////////////////////////////////////
cluster('azcrp.kusto.windows.net').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where operationName == 'VirtualMachines.ForceDelete.DELETE' or operationName =='VirtualMachines.ResourceOperation.DELETE'
| where region !contains "validation"
//|where subscriptionId == 'bad9a825-faff-4cf5-b6cf-4a4fcb1d5843' // Snowflake's sub
| summarize count(), percentiles(e2EDurationInMilliseconds / 1000, 50, 90, 99) by  operationName, region//, correlationId

cluster('azcrp.kusto.windows.net').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where operationName == 'VirtualMachines.ForceDelete.DELETE' or operationName =='VirtualMachines.ResourceOperation.DELETE'
| distinct operationName, region
| evaluate pivot(operationName)
| where "VirtualMachines.ForceDelete.DELETE" != "0"



///////////////////////////////////////////////
// Specific errors noted in the past
///////////////////////////////////////////////

// https://portal.microsofticm.com/imp/v3/incidents/omnisearch?searchString=RnmServiceDeletionCleanupTimeout
// Telefonica RNM error
cluster("Azcrp").database("crp_allprod").ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(14d)
| where resourceName startswith 'aks'
| where subscriptionId == "c5c5cab3-9fce-4c83-94f0-3dd19866f718"
| where exceptionType == "Microsoft.WindowsAzure.Networking.Nrp.Frontend.Common.NrpInternalException"
| summarize count() by operationName, userAgent

// NRP bug that blocks deletion, which affected telefonica
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(60d)
| where resourceName startswith 'aks'
| where errorDetails contains "RNM did not finish deleting resources"
| where resultCode == "NetworkingInternalOperationError"
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 


// Get disk attach while detach errors due to cloud provider bug for AKS 
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(100d)
| where resourceName startswith 'aks'
| where resultType == 1
//| where resultCode == 'ConflictingUserInput' and httpStatusCode == 409
| where errorDetails contains "A disk with name kubernetes-dynamic-pvc-" and errorDetails contains "already exists in Resource Group " 
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 

// VMProcessingHalted
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(100d) 
| where resourceName  startswith "aks"
| where resultCode == "VMProcessingHalted"
| summarize count() by bin(PreciseTimeStamp, 1d), region
| render columnchart  

//////////////////////////////////////////////////////////////////

/////////////////////////////////////////////////////////////////
// AKS VMSS non-Get one day operations
let customers =  cluster('accia').database("CommonDims").Dim_CustomerSubscription
    | distinct SubscriptionGuid, Customer, AccountTypeFlag | project-rename subscriptionId=SubscriptionGuid;
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
//| where operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| where PreciseTimeStamp > ago(1d)
| where resourceName startswith 'aks' 
| extend labelsJson = parse_json(labels)
//| extend ConvergedApiVMSS = tostring(labelsJson.ConvergedApiVmss)
//| where ConvergedApiVMSS == false
| extend r=parse_json(requestEntity) 
| extend spg=labelsJson.SinglePlacementGroup, fd=labelsJson.FaultDomainCount, p=r.properties
| project-away MonitoringApplication, Pid, Tid, operationId, clientRequestId, correlationId, serviceBuild, wFAppName, partitionId, replicaId, clientApplicationId, apiVersion, labels, SourceNamespace, SourceVersion, goalSeekingActivityId, r
| join kind= leftouter customers on subscriptionId | project-away subscriptionId1
| where AccountTypeFlag == 'External' and Customer != 'HashiCorp, Inc.' and Customer != 'CloudDirect'
| summarize count() by resultType


// AKS query on OSProvisioningTimedOut, due to Host Networking Manalox driver bug
cluster('aks').database('AKSprod').AsyncContextActivity
| where TIMESTAMP between (datetime(2020-10-10)..datetime(2020-10-30)) 
| where msg contains "OSProvisioningTimedOut"
| summarize dcount(operationID) by bin(TIMESTAMP, 1d)
| render timechart


//
// Get Infra VMSS stuck in upgrades or deletes due to VMSS bug
// Due to port exhaustion when VMSS mistakenly update Networking profile
// for deletes
// 
let err='SpecifiedAllocatedOutboundPortsForOutboundRuleExceedsTotalNumberOfAvailablePorts';
let infra='k8s-infra-';
let infra_rg="hcp-underlay-";
let aks='aks';
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp  between (datetime(2020-10-25)..now()) 
| where resourceGroupName startswith infra_rg
//| where resourceName startswith infra // 'aks'  // there is no better way to identify AKS vmss other than this for now
//| where errorDetails contains err
| where resultCode == err
| summarize count() by operationName, bin(PreciseTimeStamp, 1h)
| render timechart

////////////////////////////////////////////////////////
// Top 10 Getters
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP > ago(1d)
| where resourceName startswith 'aks' 
| where operationName contains ".GET" 
| summarize  count() by userAgent 
| sort by count_ desc 
| take 10

////////////////////////////////////////////////////////



//
// One customer VMSS call latency with vvMCountDelta, called from AKS
//
let startTime = ago(1d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where resourceName startswith 'aks'
| where subscriptionId == "7ff878e4-9faa-4fd2-b534-8bc437f4d100"
| where PreciseTimeStamp between(startTime .. endTime)
| sort by PreciseTimeStamp desc
| join kind = inner(cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where PreciseTimeStamp between(startTime .. endTime)) on $left.operationId == $right.operationId
| join 
| project PreciseTimeStamp, e2eInMin=round(e2EDurationInMilliseconds / 60000.0, 2), operationName, targetInstanceCount, vMCountDelta, requestEntity, httpStatusCode, resultCode, resultType, errorDetails, predominantErrorCode, resourceName, userAgent, resourceGroupName
| sort by e2eInMin desc



///////////////////////////////////////////////////////////////////////////////
// VMSS "NetworkingInternalOperationError" and 
//      "An unexpected error occured while processing the network profile of the VM. Please retry later."
// This result in huge spike in VMSS call latency, AKS oncall, to mitigate for customer, can only upgrade vm one by one by hand!
///////////////////////////////////////////////////////////////////////////////

// AKS: latency and count in last 1 day
let startTime = ago(1d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between(startTime .. endTime)
//| where userAgent contains "AKS-VMSS" or userAgent contains "cluster-autoscaler-aks" 
| where resourceName startswith 'aks'
| where resultCode == "NetworkingInternalOperationError"
| where errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| extend e2eInMin = round(e2EDurationInMilliseconds / 60000., 2)
| summarize percentiles(e2eInMin, 10, 50, 95, 99), max(e2eInMin), count()

// AKS: timechart by count daily in last 120 days
let startTime = ago(120d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between(startTime .. endTime)
//| where userAgent contains "AKS-VMSS" or userAgent contains "cluster-autoscaler-aks" 
| where resourceName startswith 'aks'
| where resultCode == "NetworkingInternalOperationError" and errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 

// AKS: timechart by latency percentile daily
let startTime = ago(30d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between(startTime .. endTime)
//| where userAgent contains "AKS-VMSS" or userAgent contains "cluster-autoscaler-aks" 
| where resourceName startswith 'aks'
| where resultCode == "NetworkingInternalOperationError" and errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| extend e2eInMin = round(e2EDurationInMilliseconds / 60000., 2)
| summarize percentiles(e2eInMin, 10, 50, 95, 99) by bin(PreciseTimeStamp, 1d) // , max(e2eInMin)
| render timechart 

// ALL calls in last day
let startTime = ago(1d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between(startTime .. endTime)
| where resultCode == "NetworkingInternalOperationError" and errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| extend e2eInMin = round(e2EDurationInMilliseconds / 60000., 2)
| summarize percentiles(e2eInMin, 25, 50, 95, 99), max(e2eInMin), count()

// ALL: timechart by count daily
let startTime = ago(120d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between(startTime .. endTime)
| where resultCode == "NetworkingInternalOperationError" and errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| summarize count() by bin(PreciseTimeStamp, 1d)
| render timechart 

// ALL: timechart by latency percentile daily
let startTime = ago(30d);
let endTime = now();
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
//| where PreciseTimeStamp between(startTime .. endTime)
| where resultCode == "NetworkingInternalOperationError" and errorDetails contains "An unexpected error occured while processing the network profile of the VM. Please retry later."
| extend e2eInMin = round(e2EDurationInMilliseconds / 60000., 2)
| summarize percentiles(e2eInMin, 50, 95, 99) by bin(PreciseTimeStamp, 1d) // , max(e2eInMin)
| render timechart 

//////////////////////////////////////////////////////////////////////////
// VMSS disk attach/detach count and error by day
//////////////////////////////////////////////////////////////////////////
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where resourceName startswith 'aks'  
| where operationName in ('VMScaleSetVMs.VMScaleSetVMsOperation.PUT', 'VirtualMachines.ResourceOperation.PATCH', 'VirtualMachines.ResourceOperation.PUT')
| where labels contains_cs "DiskAttached" or labels contains_cs "DiskDetached"
| where userAgent contains_cs "kubernetes-cloudprovider"
| extend l=parse_json(labels), r=parse_json(requestEntity)
| extend data=r.properties.storageProfile.dataDisks, diskOp = iff(tobool(l.DiskAttached), "attach", "detach")
| project PreciseTimeStamp, diskOp, latencyInSeconds=e2EDurationInMilliseconds/1000., region, r, data, l, userAgent, errorDetails, exceptionType, resourceName, resultType
| summarize total=count(), userErr = countif(resultType == 1), serverErr = countif(resultType == 2)  by diskOp, bin(PreciseTimeStamp, 1d)
| extend serverErrRate=serverErr*100./total, userErrRate=userErr*100./total
//| project-away total, err
| render timechart 


// VMSS disk attach, detach latency by day
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| where resultType ==0
| where resourceName startswith 'aks'  
| where labels contains_cs "DiskAttached" or labels contains_cs "DiskDetached"
| where userAgent contains_cs "kubernetes-cloudprovider"
| extend l=parse_json(labels), r=parse_json(requestEntity)
| extend data=r.properties.storageProfile.dataDisks, diskOp = iff(tobool(l.DiskAttached), "attach", "detach")
| project-away Node, MonitoringApplication, Pid, Tid, serviceBuild, wFAppName, partitionId, replicaId, apiVersion, SourceNamespace, SourceVersion, goalSeekingActivityId, labels
| project PreciseTimeStamp, diskOp, latencyInSeconds=e2EDurationInMilliseconds/1000., region, r, data, l, userAgent, errorDetails, exceptionType, resourceName
| summarize percentiles(latencyInSeconds, 50, 95, 99, 99.9), max(latencyInSeconds) by diskOp, bin(PreciseTimeStamp, 1d)
| render timechart 

/*  Do not use this one, use the one above using nonGet table,  Get disk attach, detach error rate
ApiQosEvent    // the first two lines can be replaced by ApiQosEvent_nonGet and it will be much faster
| where operationName in ('VMScaleSetVMs.VMScaleSetVMsOperation.PUT', 'VirtualMachines.ResourceOperation.PATCH', 'VirtualMachines.ResourceOperation.PUT')
| where TIMESTAMP > ago(1d)
| where resourceName startswith 'aks'  
| where labels contains_cs "DiskAttached" or labels contains_cs "DiskDetached"
| where userAgent contains_cs "kubernetes-cloudprovider"
| extend l=parse_json(labels), r=parse_json(requestEntity)
| extend data=r.properties.storageProfile.dataDisks, diskOp = iff(tobool(l.DiskAttached), "attach", "detach")
| project-away Node, RPTenant, MonitoringApplication, Pid, Tid, serviceBuild, wFAppName, partitionId, replicaId, apiVersion, SourceNamespace, SourceMoniker, SourceVersion, goalSeekingActivityId, labels
| project PreciseTimeStamp, diskOp, latencyInSeconds=e2EDurationInMilliseconds/1000., region, r, data, l, userAgent, errorDetails, exceptionType, resourceName, resultType
| summarize total=count(), err = countif(resultType == 1) by diskOp, bin(PreciseTimeStamp, 1h)
| extend errRate=err*100./total
//| project-away total, err
| render timechart 
*/

cluster('azurecm').database('AzureCM').CRP_Latency_Trends_By_Subscriptions(ago(7d), ago(0h), 99, "CrpAttachDisk", dynamic(['b9d2281e-dcd5-4dfd-9a97-0d50377cdf76'])) 
| render timechart


/////////////////////////////////////////////////////////////////////////
// Get distribution by FD
/////////////////////////////////////////////////////////////////////////
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(1d)
| where resourceName startswith 'aks'  
| where labels contains_cs "DiskAttached" or labels contains_cs "DiskDetached"
| where userAgent contains_cs "kubernetes-cloudprovider"
| extend l=parse_json(labels), r=parse_json(requestEntity)
| extend FD= toint(l.FaultDomainCount)
| project-away Node, MonitoringApplication, Pid, Tid, serviceBuild, wFAppName, partitionId, replicaId, apiVersion, SourceNamespace, SourceVersion, goalSeekingActivityId, labels
| summarize count() by FD


/////////////////////////////////////////////////////////////////////////
// JOIN ApiQosEvent and VmssQoSEvent table
/////////////////////////////////////////////////////////////////////////
let start = datetime(2020-09-02T15:00:00);
let end = datetime(2020-09-02T16:00:00);
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp between (start..end)
| where operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| where resourceName startswith 'aks'
| project subscriptionId, operationId, operationName, resourceGroupName, vmssName=resourceName, resultType, resultCode
| join kind=leftouter  (
    cluster('azcrp').database('crp_allprod').VmssQoSEvent  // This is missing some data
    | where PreciseTimeStamp between (start..end)
    | where image startswith 'PlatformImage|microsoft-aks|aks'
    | project subscriptionId, operationId, operationName, vmssName, image, oSType, targetVMSize, vMCountDelta, resultType, e2EDurationSeconds
) on subscriptionId, operationId, operationName, vmssName
| take 10

///////////////////////////////////////////////////////////////////////////
//VMO Operations
///////////////////////////////////////////////////////////////////////////

// Count of VMO operations from external customers in last day
let customers =  cluster('accia').database("CommonDims").Dim_CustomerSubscription
    | distinct SubscriptionGuid, Customer, AccountTypeFlag | project-rename subscriptionId=SubscriptionGuid;
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| extend l = parse_json(labels)
| where isnotempty(l.VmssReferenceUri)  // indicator this is VMO
| extend r=parse_json(requestEntity) 
| extend spg=l.SinglePlacementGroup, fd=l.FaultDomainCount, p=r.properties
| project-away MonitoringApplication, Pid, Tid, operationId, clientRequestId, correlationId, serviceBuild, wFAppName, partitionId, replicaId, clientApplicationId, apiVersion, labels, SourceNamespace, SourceVersion, goalSeekingActivityId, r
| join kind= leftouter customers on subscriptionId | project-away subscriptionId1
| where AccountTypeFlag == 'External' //and Customer != 'HashiCorp, Inc.' and Customer != 'CloudDirect'
| summarize count() by Customer, bin(PreciseTimeStamp, 1d)
| render timechart 

ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(10d)
| extend l = parse_json(labels)
| where isnotempty(l.VmssReferenceUri)  // indicator this is VMO
| where (l.DiskAttached == "True" or l.DiskDetached == "True") 
 | take 1

// VMO Disk attach/detach count by customers or 
// Will be good to find out per VMSS
let customers =  cluster('accia').database("CommonDims").Dim_CustomerSubscription
    | distinct SubscriptionGuid, Customer, AccountTypeFlag | project-rename subscriptionId=SubscriptionGuid;
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| extend l = parse_json(labels)
| where isnotempty(l.VmssReferenceUri)  // indicator this is VMO
| where (l.DiskAttached == "True" or l.DiskDetached == "True")   // Indicate disk attach and detach operation
| extend r=parse_json(requestEntity) 
| extend data=r.properties.storageProfile.dataDisks, diskOp = iff(tobool(l.DiskAttached), "attach", "detach"), spg=l.SinglePlacementGroup, fd=l.FaultDomainCount
//| project-away MonitoringApplication, Pid, Tid, operationId, clientRequestId, correlationId, serviceBuild, wFAppName, partitionId, replicaId, clientApplicationId, apiVersion, labels, SourceNamespace, SourceVersion, goalSeekingActivityId, r
//| project PreciseTimeStamp, diskOp, latencyInSeconds=e2EDurationInMilliseconds/1000., region, data, l, userAgent, errorDetails, exceptionType, resourceName, resultType
| join kind= leftouter customers on subscriptionId | project-away subscriptionId1
| where AccountTypeFlag == 'External' //and Customer != 'HashiCorp, Inc.' and Customer != 'CloudDirect'
//| summarize total=count(), userErr = countif(resultType == 1), serverErr=countif(resultType == 2) by diskOp, bin(PreciseTimeStamp, 1d)
| summarize count() by resourceName, bin(PreciseTimeStamp, 1d) // Customer
| render timechart 

// VMO disk attach/detach latency
ApiQosEvent_nonGet
| where PreciseTimeStamp > ago(30d)
| extend l = parse_json(labels)
| where isnotempty(l.VmssReferenceUri)  // indicator this is VMO
| where (l.DiskAttached == "True" or l.DiskDetached == "True")   // Indicate disk attach and detach operation
| extend r=parse_json(requestEntity) 
| extend data=r.properties.storageProfile.dataDisks, diskOp = iff(tobool(l.DiskAttached), "attach", "detach"), spg=l.SinglePlacementGroup, fd=l.FaultDomainCount
//| project-away MonitoringApplication, Pid, Tid, operationId, clientRequestId, correlationId, serviceBuild, wFAppName, partitionId, replicaId, clientApplicationId, apiVersion, labels, SourceNamespace, SourceVersion, goalSeekingActivityId, r
| project PreciseTimeStamp, diskOp, latencyInSeconds=e2EDurationInMilliseconds/1000., region, data, l, userAgent, errorDetails, exceptionType, resourceName, resultType
| summarize percentiles(latencyInSeconds, 50, 95, 99, 99.9), max(latencyInSeconds) by diskOp, bin(PreciseTimeStamp, 1d)
| render timechart 

let lookback = ago(60d);
let aksKnownInternalSubscriptionIds=(BlackboxMonitoringActivity
| where PreciseTimeStamp > ago(24h)
| where internalSub
| distinct subscriptionID);
let aksE2ESubscriptionIds = pack_array(
  "c4a3146e-32f0-4f42-b343-ddc5ad7b8ff6",
  "76907309-9f00-4b15-a06a-f45e789ba96c",
  "9229c7fa-9504-4c07-b727-8013960f25f1",
  "c4a3146e-32f0-4f42-b343-ddc5ad7b8ff6",
  "d1317216-c437-4369-ac7d-16a6f15a44fd");
cluster('azcrp').database('crp_allprod').ApiQosEvent_nonGet
| where PreciseTimeStamp > lookback
| where userAgent endswith "AKS-VMSS-Client"
| where subscriptionId !in (aksKnownInternalSubscriptionIds)
//| where region in ("eastus", "westus2", "westeurope")
| summarize
  allocationFailedCount=dcountif(operationId, resultCode == "AllocationFailed/ComputeAllocationInternalError"),
  totalCount=dcount(operationId)
by bin(PreciseTimeStamp, 1d), region
| extend allocationFailedRatio=toreal(allocationFailedCount) / toreal(totalCount)
| project PreciseTimeStamp, region, allocationFailedRatio
| render timechart


/////////////////////////////////////////////////////////////////


////////////////////////////////////////////////////////////////
// AKS Windows image missing issue
////////////////////////////////////////////////////////////////

// AKS Windows image not found error from ApiQosEvent table
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where PreciseTimeStamp between(datetime(8/14/2020 00:00:00)..datetime(8/15/2020 20:00:00))
| where resourceName startswith 'aks' and resourceGroupName !contains "aksrnr" 
| where (resultType == 1) and (resultCode has 'PlatformImageNotFound') and (errorDetails contains "microsoft-aks:aks-windows")
| summarize count(), dcount(subscriptionId) by bin(TIMESTAMP, 1h)
| render timechart 

cluster('azcrp').database('crp_allprod').ApiQosEvent
| where PreciseTimeStamp > ago(1h)
| where resourceName startswith 'aks' and resourceGroupName !contains "aksrnr" 
| where (resultType == 1) and (resultCode has 'PlatformImageNotFound') and (errorDetails contains "microsoft-aks:aks-windows")
//| summarize count() by subscriptionId
| project errorDetails, subscriptionId
| extend e=parse_json(errorDetails) | extend m=tostring(e.message)
| summarize count(), dcount(subscriptionId) by m


// Get ratio of Windows image missing / total requests
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where PreciseTimeStamp between(datetime(8/14/2020 00:00:00)..datetime(8/15/2020 20:00:00))
| where resourceName startswith 'aks' and resourceGroupName !contains "aksrnr" 
| summarize total=count(), imageError=countif((resultType == 1) and (resultCode has 'PlatformImageNotFound') and (errorDetails contains "microsoft-aks:aks-windows"))
| extend r = imageError*100.0/total

// AKS Windows image missing issue on ARM logs
cluster('armprod').database('ARMProd').EventServiceEntries
| where PreciseTimeStamp between(datetime(8/14/2020 00:00:00)..datetime(8/15/2020 20:00:00))
| where resourceProvider == "Microsoft.Compute"
| where properties contains"The platform image 'microsoft-aks:aks-windows:" and properties contains "PlatformImageNotFound"
| where subscriptionId !in ("8ecadfc9-d1a3-4ea4-b844-0d9f87e4d7c8", 
 "359833f5-8592-40b6-8175-edc664e2196a", 
 "c1089427-83d3-4286-9f35-5af546a6eb67",
 "5299e6b7-b23b-46c8-8277-dc1147807117",
 "5ea8beca-77b8-44cb-8871-93620f04a6e7",
 "12c3ab6c-eb5f-4378-b9b0-d587b7665766",
 "c0f60687-8f09-4186-801b-9dd11d82d2e1",
 "6a1e7c96-24b8-4880-8263-9732eb76b849",
 "6cedb63e-a5a2-4d1b-bf27-71f3688871ee",
 "5ce1ccad-10d3-4d04-a455-4ab42ee64a61",
 "990d87fa-2d5a-48cc-bdff-0d3c6b9dd32d",
 "54aea2c5-5855-40ac-a5e7-5f3252640fae",
 "9229c7fa-9504-4c07-b727-8013960f25f1",
 "692aea0b-2d89-4e7e-ae30-fffe40782ee2",
 "86501df2-2fc5-4fc3-b193-d305572e416f",
 "4b27da8c-09a3-46df-aafb-da7f74e0b610",
 "d1317216-c437-4369-ac7d-16a6f15a44fd",
 "76907309-9f00-4b15-a06a-f45e789ba96c",
 "2b915285-1302-4a0f-8b54-f502a6b619a6",
 "c4a3146e-32f0-4f42-b343-ddc5ad7b8ff6")
| summarize distinctSub=dcount(subscriptionId), totalOperation=count()// by operationName


////////////////////////////////////////////////////////////////////////////////////////
// AKS VMSS CREATE
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP > ago(1h)
| where resourceName startswith 'aks' //and resourceGroupName startswith 'mc_'
| where operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| take 10
| join cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where PreciseTimeStamp > ago(1h)
//| where MonitoringApplication contains ""
//| where subscriptionId == "" and resourceGroupName == "" and vmssName contains ""
| where image contains "aks"
| distinct image
//| where errorDetails contains "'vmssCSE'" and errorDetails contains "status="
//| extend status = extract("status=([0-9.]+)", 1, errorDetails)
//| project resourceName, resourceGroupName, operationName
//| summarize nomc=countif(resourceGroupName !startswith 'mc_'), count()


///////////////////////////////////////////////////////////////
// Cluster Auto Scaler errors
cluster("Azcrp").database("crp_allprod").ApiQosEvent 
| where TIMESTAMP > ago(1d)
| where operationName !contains ".GET"
| where resourceName startswith 'aks'
| where userAgent contains 'cluster-autoscaler-aks'
| extend casVersion = extract("cluster-autoscaler-aks/(.+)", 1, userAgent)
| summarize count() by resultType, casVersion
| sort by casVersion
| render columnchart  with ( kind=stacked , xcolumn=casVersion)


// Cloud provider errors
cluster("Azcrp").database("crp_allprod").ApiQosEvent 
| where TIMESTAMP > ago(1d)
| where operationName !contains ".GET"
| where resourceName startswith 'aks'
| where userAgent contains 'kubernetes-cloudprovider'
| extend version = extract("kubernetes-cloudprovider/(.+)", 1, userAgent)
| summarize count() by resultType, version
| sort by version
| render columnchart  with ( kind=stacked , xcolumn=version)

////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////
// Get regions AKS is in
cluster("Azcrp").database("crp_allprod").ApiQosEvent 
| where TIMESTAMP > ago(1d)
| where resourceName startswith 'aks'  
| distinct region
////////////////////////////////////////////////////////////////////








// FROM ARM
// Get all PUT and DELETE VMSS calls for a sub and AKS node pool 
let start = datetime(2020-03-19 04:30:00);
let end = datetime(2020-03-19 07:00:00);
union cluster('armprod').database('ARMProd').HttpIncomingRequests
| where TIMESTAMP between (start .. end)
| where httpStatusCode <> -1
| where operationName == 'POST/SUBSCRIPTIONS/RESOURCEGROUPS/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINESCALESETS/DELETE' 
    or operationName == 'PUT/SUBSCRIPTIONS/RESOURCEGROUPS/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINESCALESETS/'
| where subscriptionId == "d3ab5211-edb0-4382-9346-f500714e236c"
| where targetUri contains 'aks-service-52871592-vmss'
//| summarize count() by operationName



// DEBUG DEBUG DEBUG one customer subscription and cluster

// The following queries helped investigating VMSS throttling issue due to queries from Cloud Provider

// 1. START FROM ARM HttpIncomingRequests, count by status code
let start = datetime('2020-02-07 00:00');
let end = datetime('2020-02-11 00:00');
cluster('armprod').database('ARMProd').HttpIncomingRequests
| where TIMESTAMP between (start .. end)
| where subscriptionId has 'bbad3705-bf2b-4e41-a668-38f765169a99'
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'PUT/SUBSCRIPTIONS/RESOURCEGROUPS/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINESCALESETS/VIRTUALMACHINES/'
| where httpStatusCode != -1
| summarize _ = count() by httpStatusCode

// 2. USE CRP  
// VMSS PUT calls from cloud provider every 10 seconds
let start = datetime('2020-02-07 00:00');
let end = datetime('2020-02-11 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99"  and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS of Customer's subid
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| make-series count() default = 0 on TIMESTAMP in range(start, now(), 10s)
| render timechart    

// VMSS PUT calls from cloud provider every 10 seconds
let start = datetime('2020-02-07 00:00');
let end = datetime('2020-02-11 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99"  and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS of Customer's subid
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| extend updateNetwork = not(isnull(e_properties_networkProfileConfiguration)), updateStorage = not(isnull(e_properties_storageProfile))
| extend lbBackendPools = e_properties_networkProfileConfiguration.networkInterfaceConfigurations[0].properties.ipConfigurations[0].properties.loadBalancerBackendAddressPools
| extend lbCount = array_length(lbBackendPools)
| project TIMESTAMP, instanceId, httpStatusCode, lbCount, lbBackendPools, resourceName, resultType, e
//| make-series count() default = 0 on TIMESTAMP in range(start, now(), 1h) by lbCount
| summarize noILB=countif(lbCount==2), withILB=countif(lbCount==3) by bin(TIMESTAMP, 10s)
| render timechart            

// Get unique network configurations in each PUT
// Turned out to be just 2, the only difference is the second one added Internal Load Balancer
// And these two overlap in time
let start = datetime('2020-02-06 00:00');
let end = datetime('2020-02-15 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS in customer's sub
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| extend updateNetwork = not(isnull(e_properties_networkProfileConfiguration)), updateStorate = not(isnull(e_properties_storageProfile))
| project TIMESTAMP, instanceId, updateNetwork, updateStorate, httpStatusCode, e_properties_networkProfileConfiguration, resourceName, resultType, e
| summarize min(TIMESTAMP), max(TIMESTAMP), count() by tostring(e_properties_networkProfileConfiguration)
| extend nowTime = now()

// Get detailed calls to VMSS PUT from Azure Cloud Provider
// sorted by time
let start = datetime('2020-02-06 00:00');
let end = datetime('2020-02-15 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS in customer's sub
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| extend updateNetwork = not(isnull(e_properties_networkProfileConfiguration)), updateStorage = not(isnull(e_properties_storageProfile))
| extend lbBackendPools = e_properties_networkProfileConfiguration.networkInterfaceConfigurations[0].properties.ipConfigurations[0].properties.loadBalancerBackendAddressPools
| extend lbCount = array_length(lbBackendPools)
| project TIMESTAMP, instanceId, httpStatusCode, lbCount, lbBackendPools, resourceName, resultType, e
| sort by TIMESTAMP asc
    

let start = datetime('2020-02-06 00:00');
let end = datetime('2020-02-15 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS in customer's sub
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| project TIMESTAMP, instanceId, httpStatusCode,  e_properties_networkProfileConfiguration, resourceName, resultType
| sort by TIMESTAMP asc 

let start = datetime('2020-02-06 00:00');
let end = datetime('2020-02-15 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS in customer's sub
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| summarize arg_max(TIMESTAMP, *) by instanceId
| project TIMESTAMP, resourceName, resultType, httpStatusCode, instanceId, e_properties_networkProfileConfiguration


// Exam Cloud provider calls for one VM instance in a VMSS
let start = datetime('2020-02-08 00:00');
let end = datetime('2020-02-11 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" and resourceName startswith "aks-nodepool1-85069547-vmss/" // Focus on 1 VMSS in customer's sub
| where userAgent contains 'kubernetes-cloudprovider/v1.15.7' and operationName == 'VMScaleSetVMs.VMScaleSetVMsOperation.PUT'
| extend e= todynamic(requestEntity)
| extend instanceId = tostring(e.instanceId), e.properties, e.properties.networkProfileConfiguration, e.properties.storageProfile
| project TIMESTAMP, resourceName, resultType, httpStatusCode, instanceId, e_properties_networkProfileConfiguration
//| where instanceId == 0
| summarize (minTime, minHttpStatusCode) = arg_min(TIMESTAMP, httpStatusCode), (maxTime, maxHttpStatusCode) = arg_max(TIMESTAMP, httpStatusCode) 
    by instanceId, tostring(e_properties_networkProfileConfiguration)
| sort by instanceId asc

// Get all CRP activities on a sub within time range.
// Q 1: Why many calls without userAgent?
// Q 2: Why some no resourceName? And some resourceName is hex value?
let start = datetime('2020-02-08 00:00');
let end = datetime('2020-02-10 00:00');
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (start .. end)
| where subscriptionId == "bbad3705-bf2b-4e41-a668-38f765169a99" // Cx sub
| where userAgent contains 'kubernetes-cloudprovider'  // Narrow down to calls only from K8S cloud provider
| where operationName !startswith 'Operations.'
| summarize min(TIMESTAMP), max(TIMESTAMP), count() by resourceName, operationName, userAgent

// Get CRP calls from an AKS cluster
cluster('azcrp').database('crp_allprod').ApiQosEvent
| where TIMESTAMP between (datetime('2020-01-29')..datetime('2020-01-30'))
| where subscriptionId == "d5676c2d-52cd-405a-b626-42b157652913"  //"be90498f-7d29-484c-a606-cabe0c55e352"
| project TIMESTAMP, operationName, resourceGroupName, region, userAgent


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// Use VmssQosEvent for latency data
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


// Get unique AKS images
cluster('azcrp').database('crp_allprod').VmssQoSEvent | where image contains "aks" 
| summarize count() by image
| sort by count_ desc


// Get Disk attach detach performance from VMSS
cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where TIMESTAMP > ago(3d)
| where image startswith "PlatformImage|microsoft-aks|aks"
//and subscriptionId == "c194ceb5-a5a5-43bc-ac99-2e0cd08859fd"
| join (cluster('azcrp').database('crp_allprod').ApiQosEvent 
        | where TIMESTAMP > ago(30d)
        | where resultType ==0
        | where labels contains "attach" or labels contains "detach"
        ) 
on  subscriptionId, operationId
| extend opName = iff(labels contains "attach","AttachDisk","DetachDisk")
| project TIMESTAMP, opName, e2EDurationSeconds, resourceGroupName, vmssName
| summarize percentiles(e2EDurationSeconds, 50, 95, 99, 99.9), max(e2EDurationSeconds), count() by opName, bin(TIMESTAMP, 1h)
| render timechart 

// WARNING !!  MutatingComponentQoSTable is empty
// AKS latency in VMSS by stage
// WARNING !!
let starttime = ago(1h);
let vmssqos = cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where PreciseTimeStamp >= ago(1h)
| where image startswith "PlatformImage|microsoft-aks|aks" and oSType == 'Linux'
//| where isNewVMScaleSetOperation == true
| project activityId=operationId;
let mutateqos = cluster('azcrp').database('crp_allprod').MutatingComponentQoSTable
| where PreciseTimeStamp >= ago(30d)
| where operationName =="PollForVMExtensionsProvisioningResult" or operationName =="PollForRoleInstanceStatus" or operationName =="PollForRoleInstanceProvisioningResult"
| project operationName, durationInSec=durationInMs/1000.0, activityId;
vmssqos |join mutateqos on activityId
| summarize percentiles(durationInSec,80, 90, 99) by operationName


// VMSS non-delete operation latency
cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where TIMESTAMP > ago(30d)
| where oSType == 'Linux'
| where operationName !contains "delete" and resultType == 0 // successful non-delete
| extend isGPU = iff(targetVMSize startswith "Standard_N", true, false), isAKS = iff(image startswith "PlatformImage|microsoft-aks|aks", true, false)
| summarize percentiles(e2EDurationSeconds, 50, 90, 99) by isAKS, isGPU, day = bin(TIMESTAMP, 1d)
| project day, isAKS, isGPU, p50=percentile_e2EDurationSeconds_50, p90=percentile_e2EDurationSeconds_90, p99=percentile_e2EDurationSeconds_99
| sort by day, isGPU, isAKS
//| render timechart 


/////////////////////////////////////////////////////////////////////////////////////
// DEBUG AKS errors from VmssQosEvents
// AKS failure rate is alarmingly higher than non-AKS
// Comment out summary statements to get error details
/////////////////////////////////////////////////////////////////////////////////////

// Create function in AKSprod
.create-or-alter function with (
  folder = "qike", 
  docstring = "Join CRP ApiQosEvent and VmssQoSEvent. Return columns: \n  TIMESTAMP, resultType, subscriptionId, operationId, operationName, isAKS, isGPU, predominantErrorCode, predominantErrorDetail, predominantErrorCount, agent", 
  skipvalidation = "false")
getCrpQosAndErrors(starttime: datetime, endtime : datetime) {
//let endtime = endofday(ago(1d));
//let starttime = endtime - 1h;
let apiQoS=cluster('azcrp').database('crp_allprod').ApiQosEvent
        | where TIMESTAMP >= starttime and TIMESTAMP < endtime
        | project subscriptionId, operationId, agent=case(
                userAgent contains 'cluster-autoscaler', 'CA'
                , userAgent contains 'kubernetes-cloudprovider', 'CP'
                , userAgent contains 'Azure-SDK-For-Python', 'CLI'
                , userAgent contains 'AzurePowershell', 'CLI',  userAgent);
let vmssQoS=cluster('azcrp').database('crp_allprod').VmssQoSEvent
        | where TIMESTAMP >= starttime and TIMESTAMP < endtime
        | where oSType == 'Linux'
        | extend isAKS = iff(image startswith "PlatformImage|microsoft-aks|aks", true, false), isGPU = iff(targetVMSize startswith "Standard_N", true, false)
        | project TIMESTAMP, resultType, subscriptionId, operationId, operationName, isAKS, isGPU, predominantErrorCode, predominantErrorDetail, predominantErrorCount
          //, resourceGroupName, vmssName, image, availabilitySetCount, targetInstanceCount, targetVMSize, vMCountDelta, vMInstanceFailureCount, extraVMInstanceOverProvisionedCount
        ;
vmssQoS | join apiQoS on  subscriptionId, operationId
| extend err=iff(predominantErrorDetail != "", toobject(predominantErrorDetail), toobject(""))
| extend errCode=tostring(err.code), errMsg=tostring(err.message), errInner = tostring(err.innererror)
| extend error=case(predominantErrorCode in ('SubnetIsFull'
                                             , 'CannotAddOrRemoveNetworkInterfaceConfigsFromARunningVMScaleSet'
                                             , 'InvalidResourceReference'
                                             , 'PrimaryNetworkInterfaceConfigurationCannotBeChangedOnVMScaleSet'
                                             , 'PublicIPCountLimitExceededByVMScaleSet'), predominantErrorCode, 
                    errInner == "",  predominantErrorCode,
                    errInner)
| project-away subscriptionId1, operationId1, err
}


//
// Get and compare AKS and non-AKS error rate from VMSSQoSEvents table
//
let endtime = endofday(ago(1d));
let starttime = endtime - 1h;
cluster('aks').database('AKSprod').getCrpQosAndErrors(starttime, endtime)
| summarize total = count(), err1=countif(resultType == 1), err2 = countif(resultType ==2) by isAKS//, agent, operationName, predominantErrorCode, errInner//, day = bin(TIMESTAMP, 1d)
| extend err1Rate = err1 * 1.0 / total, err2Rate = err2 * 1.0 / total
| sort by total desc
//| render timechart with (xcolumn=day, ycolumns=count_, series= agent, resultType) 


let endtime = endofday(ago(1d));
let starttime = endtime - 1h;
cluster('aks').database('AKSprod').getCrpQosAndErrors(starttime, endtime)
| where isAKS == True and resultType == 1
| summarize count() by error, agent //, predominantErrorCode
| sort by count_ desc
//| render piechart 


let endtime = endofday(ago(1d));
let duration = 10d;
let subId = "c1089427-83d3-4286-9f35-5af546a6eb67";  // AKS test sub
//let rg = "gputestproductionimage";
let apiQoS=cluster('azcrp').database('crp_allprod').ApiQosEvent
        | where TIMESTAMP >= endtime-duration and TIMESTAMP < endtime
        | where subscriptionId == subId  //  and resourceGroupName == rg
        | project subscriptionId, operationId, agent=case(
                userAgent contains 'cluster-autoscaler', 'CA'
                , userAgent contains 'kubernetes-cloudprovider', 'CP'
                , userAgent contains 'Azure-SDK-For-Python', 'CLI'
                , userAgent contains 'AzurePowershell', 'CLI',  userAgent);
let vmssQoS=cluster('azcrp').database('crp_allprod').VmssQoSEvent
        | where TIMESTAMP >= endtime-duration and TIMESTAMP < endtime
        | where oSType == 'Linux'
        | where subscriptionId == subId  //  and resourceGroupName == rg
        | extend isAKS = iff(image startswith "PlatformImage|microsoft-aks|aks", true, false), isGPU = iff(targetVMSize startswith "Standard_N", true, false)
        | project TIMESTAMP, resultType, subscriptionId, operationId, operationName, isAKS, isGPU, predominantErrorCode, predominantErrorDetail, predominantErrorCount
          , resourceGroupName, vmssName, image, availabilitySetCount, targetInstanceCount, targetVMSize, vMCountDelta, vMInstanceFailureCount, extraVMInstanceOverProvisionedCount
        ;
vmssQoS | join apiQoS on  subscriptionId, operationId
| extend err=iff(predominantErrorDetail != "", toobject(predominantErrorDetail), toobject(""))
| extend errCode=tostring(err.code), errMsg=tostring(err.message), errInner = tostring(err.innererror)
| extend error=case(predominantErrorCode in ('SubnetIsFull'
                                             , 'CannotAddOrRemoveNetworkInterfaceConfigsFromARunningVMScaleSet'
                                             , 'InvalidResourceReference'
                                             , 'PrimaryNetworkInterfaceConfigurationCannotBeChangedOnVMScaleSet'
                                             , 'PublicIPCountLimitExceededByVMScaleSet'), predominantErrorCode 
                    , errInner == "",  predominantErrorCode
                    , errInner)
| project-away subscriptionId1, operationId1, err
| summarize total = count(), err1=countif(resultType == 1), err2 = countif(resultType ==2) by isGPU//, agent, predominantErrorCode, errInner
| extend err1Rate = err1 * 1.0 / total, err2Rate = err2 * 1.0 / total




/////////////////////////////////////////////////////////////////////////////////////
// Get failed VMSS operations for AKS but resultType is 0
// These succeeded with over provisioning or retries
/////////////////////////////////////////////////////////////////////////////////////
cluster('azcrp').database('crp_allprod').VmssQoSEvent
| where PreciseTimeStamp >= ago(7d)
| where oSType == 'Linux'
| extend isGPU = iff(targetVMSize startswith "Standard_N", true, false), isAKS = iff(image startswith "PlatformImage|microsoft-aks|aks", true, false)
| where isAKS
| where predominantErrorCode != "" and resultType == 0
| extend err=toobject(predominantErrorDetail)
| extend errCode=tostring(err.code), errMsg=tostring(err.message), errInner=tostring(err.innererror)
| project TIMESTAMP, err, predominantErrorCount, predominantErrorCode, predominantExceptionType, resultType,
    subscriptionId, operationId, operationName, resourceGroupName, vmssName, image,  
    availabilitySetCount, targetInstanceCount, targetVMSize, vMCountDelta, vMInstanceFailureCount, extraVMInstanceOverProvisionedCount





////////////////////////////////////////////////////////////////
// PPS count and failures and latency
///////////////////////////////////////////////////////////////
cluster('azcrp').database('crp_allprod').VMApiQosEvent
| where PreciseTimeStamp > ago(7d)
| extend usespps =  preprovisionedVMReuse contains "ReusingPreprovisionedVM"
| where resourceGroupName startswith "MC_"
| where resourceName startswith "aks-"
| summarize dcount(resourceGroupName) by usespps


cluster('azcrp').database('crp_allprod').VMApiQosEvent
| where PreciseTimeStamp > ago(7d)
| extend usespps =  preprovisionedVMReuse contains "ReusingPreprovisionedVM"
| where resourceGroupName startswith "MC_"
| where resourceName startswith "aks-"
| summarize dcount(resourceGroupName) by usespps

cluster('azcrp').database('crp_allprod').VMApiQosEvent
| where PreciseTimeStamp > ago(30d)
| where resourceName startswith 'aks'
| extend usespps =  preprovisionedVMReuse contains "ReusingPreprovisionedVM"
| summarize count() by resultType, usespps, d=bin(PreciseTimeStamp, 1d)
| render timechart with ( xcolumn=d, ycolumns=count_, series=resultType,usespps)

cluster('azcrp').database('crp_allprod').VMApiQosEvent
| where PreciseTimeStamp > ago(30d)
| where resourceName startswith 'aks'
| extend usespps =  preprovisionedVMReuse contains "ReusingPreprovisionedVM"
| summarize s=countif(resultType ==0), e1=countif(resultType ==1), e2=countif(resultType ==2) by usespps, d=bin(PreciseTimeStamp, 1d)
| extend r1=e1*100./s, r2=e2*100./s
| render timechart with ( xcolumn=d, ycolumns=r1,r2, series=usespps)






/////////////////////////////////////////////////////////////////
// Get update tenant operation latency
cluster("Azcrp").database("crp_allprod").ComponentQoSEvent
| where TIMESTAMP between(ago(10d)..now())
| where operationName  == "UpdateTenant"
| where SourceMoniker  == "Crp_MdsCrpuse2" 
| summarize latency_Seconds=percentile(durationInMs/1000, 95) by bin(TIMESTAMP, 1d)
| render timechart


/////////////////////////////////////////////////////////////
// Most of the HPC VM sizes (HBv2, HB, HC, H16r, H16mr, A8 and A9) feature a network interface for remote direct memory access (RDMA) connectivity. 
// Selected N-series sizes designated with 'r' (ND40rs_v2, ND24rs, NC24rs_v3, NC24rs_v2 and NC24r) are also RDMA-capable. 
// This interface is in addition to the standard Azure network interface available in the other VM sizes.
VMScaleSetModel
| where TIMESTAMP > ago(2h)
| where VMSize contains "HB" or VMSize contains "HC" or VMSize contains "H16r" or VMSize contains "H16mr" 
  or VMSize contains "A8" or VMSize contains "A9" 
  or VMSize contains "ND40rs_v2" or VMSize contains "ND24rs" or VMSize contains "NC24rs_v3" or VMSize contains "NC24rs_v2" or VMSize contains "NC24r"
| join (VMScaleSet
| where TIMESTAMP > ago(2h)) on SubscriptionId, ResourceGroupName, VMScaleSetId, VMScaleSetName
| distinct VMScaleSetId, LargeScaleEnabled
| summarize MPG= countif(LargeScaleEnabled =="True"), SPG=countif(LargeScaleEnabled =="False")

VMScaleSetModel
| where TIMESTAMP > ago(2h)
| where VMSize contains "HB" or VMSize contains "HC" or VMSize contains "H16r" or VMSize contains "H16mr" 
  or VMSize contains "A8" or VMSize contains "A9" 
  or VMSize contains "ND40rs_v2" or VMSize contains "ND24rs" or VMSize contains "NC24rs_v3" or VMSize contains "NC24rs_v2" or VMSize contains "NC24r"
| join (VMScaleSet
| where TIMESTAMP > ago(2h)) on SubscriptionId, ResourceGroupName, VMScaleSetId, VMScaleSetName
| distinct VMScaleSetId, LargeScaleEnabled, VMSize
| where LargeScaleEnabled =="True"
| summarize count() by VMSize | sort by VMSize


VMScaleSetModel
| where TIMESTAMP > ago(2h)
| distinct VMSize
| where VMSize contains "HB" or VMSize contains "HC" or VMSize contains "H16r" or VMSize contains "H16mr" 
  or VMSize contains "A8" or VMSize contains "A9" 
  or VMSize contains "ND40rs_v2" or VMSize contains "ND24rs" or VMSize contains "NC24rs_v3" or VMSize contains "NC24rs_v2" or VMSize contains "NC24r"

////////////////////////////////////////////////////////////////////////

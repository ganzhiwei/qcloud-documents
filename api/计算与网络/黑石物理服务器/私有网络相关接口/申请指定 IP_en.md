## 1. API Description
 
This API (RegisterBatchIp) is used to specify subnet IP for IP application.  
Domain name for API request: bmvpc.api.qcloud.com



 

## 2. Input Parameters
 The following request parameter list only provides API request parameters. Common request parameters are also needed when the API is called. For more information, please see <a href="/doc/api/372/4153" title="Common Request Parameters">Common Request Parameters</a> page. The Action field for this API is RegisterBatchIp.

| Parameter Name | Required | Type | Description |
|---------|---------|---------|---------|
| vpcId | Yes | String | VPC ID assigned by the system, which can be vpcId or unVpcId. unVpcId is recommended. For example: vpc-kd7d06of. You can query this through API DescribeBmVpcEx.  |
| subnetId | Yes | String | VPC Subnet ID assigned by the system. Both subnetId and unSubnetId are supported. unSubnetId is recommended. For example: subnet-k20jbhp0. You can query this through API DescribeBmSubnetEx.  |
| ipList | Yes | Array | Array of IP applied for. Array value range: 1-20.  |
| ipClass | No | Int | IP type. 0: CPM IP; 1: VM IP; 2: Hosting machine IP. Default is 1. |


 

## 3. Output Parameters

| Parameter Name | Type | Description |
|---------|---------|---------|
| code | Int | Common error code. 0: Successful; other values: Failed. For more information, please see <a href="https://cloud.tencent.com/doc/api/372/%E9%94%99%E8%AF%AF%E7%A0%81#1.E3.80.81.E5.85.AC.E5.85.B1.E9.94.99.E8.AF.AF.E7.A0.81" title="Common Error Codes">Common Error Codes</a> on the Error Codes page. |
| message | String | Module error message description depending on API. |
| count | Int | Number of IPs registered successfully. |
| extramsg | String | Error message returned from this API. |
| data.n | Array | Array of IPs registered successfully. |


## 4. Error Codes

 | Error Code | Error Message | Description |
|---------|---------|---------|
| -3047 |InvalidBmVpc.NotFound| Invalid VPC. VPC resource does not exist. Please verify whether the resource information entered is correct.  |
| -3030  |InvalidBmSubnet.NotFound| Invalid subnet. Subnet resource does not exist. Please verify whether the resource information entered is correct.  |
| -3031 |AvailableIpUseUp| No IPs available for assignment.  |
| -3001 | InvalidInputParams | Invalid parameter

## 5. Example
 
Input
```

  https://vpc.api.qcloud.com/v2/index.php?Action=RegisterBatchIp
	&<Common Request Parameters>
	&vpcId=vpc-2ari9m7h
	&subnetId=subnet-keqt3oty
	&ipList.0=10.1.1.2&ipList.1=10.1.1.300
```

Output
```

{
    "code": 0,
    "message": "",
    "codeDesc": "Success",
    "data": [
        "10.6.100.40"
    ],
    "extramsg":"10.1.1.300 is invalid ip."
    "count": 1
}

```



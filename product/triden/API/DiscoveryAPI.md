# 注册中心集成

> 作者 于胜强

## 注册应用实例（首次）

`curl -H "Content-Type:application/json" -X POST --data '{payload}'  http://localhost/eureka/apps/{appID}`

状态码：成功 -> 204

`appID` : 应用ID

`payload`: JSON

```json

{
	"instance": {
		"instanceId": "{instanceId}",
		"hostName": "instance0.application0.com",
		"app": "{appID}",
		"ipAddr": "20.0.0.0",
		"status": "UP",
		"overriddenstatus": "UNKNOWN",
		"port": {
			"$": 50000,
			"@enabled": "false"
		},
		"securePort": {
			"$": 50000,
			"@enabled": "true"
		},
		"countryId": 1,
		"dataCenterInfo": {
			"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
			"name": "MyOwn"
		},
		"leaseInfo": {
			"renewalIntervalInSecs": 30,
			"durationInSecs": 90,
			"registrationTimestamp": 1534841554176,
			"lastRenewalTimestamp": 1534841554181,
			"evictionTimestamp": 0,
			"serviceUpTimestamp": 1534841554171
		},
		"metadata": {
			"instanceId": "dfss-service:50000",
			"weight": "1",
			"trackSampling": "0.1",
			"management.context-path": "/dfssservice"
		},
		"homePageUrl": "http://instance0.application0.com:8080/homepage",
		"statusPageUrl": "http://instance0.application0.com:8080/status",
		"healthCheckUrl": "http://instance0.application0.com:8080/healthcheck",
		"secureHealthCheckUrl": "https://instance0.application0.com:8081/healthcheck",
		"vipAddress": "application0:8080",
		"secureVipAddress": "application0:8081",
		"isCoordinatingDiscoveryServer": false,
		"lastUpdatedTimestamp": 1534841554081,
		"lastDirtyTimestamp": 1534841554081,
		"actionType": "ADDED",
		"asgName": "application0ASG"
	}
}

```
`instanceId`: 应用实例ID

## 撤销应用实例注册

`curl -X DELETE http://localhost:8761/eureka/apps/{appID}/{instanceID}`

状态码：成功 -> 200

`appId`：应用ID

`instanceId`：应用实例ID

## 续约（心跳）

`curl -X PUT http://localhost:8761/eureka/apps/{appID}/{instanceID}`

状态码：成功 -> 200

`appId`：应用ID

`instanceId`：应用实例ID

## 查询注册实例信息

### 查询所有注册实例信息

`curl -H "Accept:application/json" -X GET http://localhost:8761/eureka/apps`

状态码：成功 -> 200

返回结果：
```json
{
	"applications": {
		"versions__delta": "1",
		"apps__hashcode": "UP_8_",
		"application": [{
			"name": "TRACE-SERVER",
			"instance": [{
				"instanceId": "oneitom-trace::trace-server:8763",
				"hostName": "localhost",
				"app": "TRACE-SERVER",
				"ipAddr": "localhost",
				"status": "UP",
				"overriddenstatus": "UNKNOWN",
				"port": {
					"$": 8763,
					"@enabled": "true"
				},
				"securePort": {
					"$": 443,
					"@enabled": "false"
				},
				"countryId": 1,
				"dataCenterInfo": {
					"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
					"name": "MyOwn"
				},
				"leaseInfo": {
					"renewalIntervalInSecs": 10,
					"durationInSecs": 30,
					"registrationTimestamp": 1540538468505,
					"lastRenewalTimestamp": 1540782054974,
					"evictionTimestamp": 0,
					"serviceUpTimestamp": 1540538467997
				},
				"metadata": {
					"instanceId": "oneitom-trace::trace-server:8763",
					"iplatformtype": "平台服务",
					"weight": "1",
					"trackSampling": "0.1",
					"management.context-path": ""
				},
				"homePageUrl": "http://localhost:8763",
				"statusPageUrl": "http://localhost:8763/info",
				"healthCheckUrl": "http://localhost:8763/health",
				"secureHealthCheckUrl": "http://localhost:8763/health",
				"vipAddress": "trace-server",
				"secureVipAddress": "trace-server",
				"isCoordinatingDiscoveryServer": "false",
				"lastUpdatedTimestamp": "1540538468505",
				"lastDirtyTimestamp": "1540538467839",
				"actionType": "ADDED"
			}]
		},
		{
			"name": "DISCOVERY-SERVER",
			"instance": [{
				"instanceId": "oneitom-discovery::discovery-server:8761",
				"hostName": "localhost",
				"app": "DISCOVERY-SERVER",
				"ipAddr": "localhost",
				"status": "UP",
				"overriddenstatus": "UNKNOWN",
				"port": {
					"$": 8761,
					"@enabled": "true"
				},
				"securePort": {
					"$": 443,
					"@enabled": "false"
				},
				"countryId": 1,
				"dataCenterInfo": {
					"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
					"name": "MyOwn"
				},
				"leaseInfo": {
					"renewalIntervalInSecs": 10,
					"durationInSecs": 30,
					"registrationTimestamp": 1540537710897,
					"lastRenewalTimestamp": 1540782052632,
					"evictionTimestamp": 0,
					"serviceUpTimestamp": 1540537690551
				},
				"metadata": {
					"instanceId": "oneitom-discovery::discovery-server:8761",
					"iplatformtype": "平台服务",
					"weight": "1",
					"configPath": "config",
					"trackSampling": "0.1",
					"management.context-path": ""
				},
				"homePageUrl": "http://localhost:8761",
				"statusPageUrl": "http://localhost:8761/info",
				"healthCheckUrl": "http://localhost:8761/health",
				"secureHealthCheckUrl": "http://localhost:8761/health",
				"vipAddress": "discovery-server",
				"secureVipAddress": "discovery-server",
				"isCoordinatingDiscoveryServer": "true",
				"lastUpdatedTimestamp": "1540537710897",
				"lastDirtyTimestamp": "1540537561315",
				"actionType": "ADDED"
			}]
		},
		... 省略 ...
        ]
	}
}
```

### 查询指定应用ID的实例信息

`curl -H "Accept:application/json" -X GET http://localhost:8761/eureka/apps/{appID}`

状态码：成功 -> 200

`appID`: 应用ID

返回信息：
```json
{
	"application": {
		"name": "DFSS-SERVICE",
		"instance": [{
			"instanceId": "oneitom-dfss::dfss-service:50000",
			"hostName": "localhost",
			"app": "DFSS-SERVICE",
			"ipAddr": "localhost",
			"status": "UP",
			"overriddenstatus": "UNKNOWN",
			"port": {
				"$": 50000,
				"@enabled": "true"
			},
			"securePort": {
				"$": 443,
				"@enabled": "false"
			},
			"countryId": 1,
			"dataCenterInfo": {
				"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
				"name": "MyOwn"
			},
			"leaseInfo": {
				"renewalIntervalInSecs": 10,
				"durationInSecs": 30,
				"registrationTimestamp": 1540538239969,
				"lastRenewalTimestamp": 1540782006436,
				"evictionTimestamp": 0,
				"serviceUpTimestamp": 1540538239463
			},
			"metadata": {
				"instanceId": "oneitom-dfss::dfss-service:50000",
				"iplatformtype": "平台服务",
				"weight": "1",
				"trackSampling": "0.1",
				"management.context-path": "/dfssservice"
			},
			"homePageUrl": "http://localhost:50000/dfssservice",
			"statusPageUrl": "http://localhost:50000/dfssservice/info",
			"healthCheckUrl": "http://localhost:50000/dfssservice/health",
			"secureHealthCheckUrl": "http://localhost:50000/dfssservice/health",
			"vipAddress": "dfss-service",
			"secureVipAddress": "dfss-service",
			"isCoordinatingDiscoveryServer": "false",
			"lastUpdatedTimestamp": "1540538239969",
			"lastDirtyTimestamp": "1540538239434",
			"actionType": "ADDED"
		}]
	}
}
```

### 查询指定实例ID的实例信息

`curl -H "Accept:application/json" -X GET http://localhost:8761/eureka/apps/{appID}/{instanceId}`

状态码：成功 -> 200

`appID`: 应用ID

`instanceId`: 应用实例ID

返回信息：
```json
{
	"instance": {
		"instanceId": "oneitom-dfss::dfss-service:50000",
		"hostName": "localhost",
		"app": "DFSS-SERVICE",
		"ipAddr": "localhost",
		"status": "UP",
		"overriddenstatus": "UNKNOWN",
		"port": {
			"$": 50000,
			"@enabled": "true"
		},
		"securePort": {
			"$": 443,
			"@enabled": "false"
		},
		"countryId": 1,
		"dataCenterInfo": {
			"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
			"name": "MyOwn"
		},
		"leaseInfo": {
			"renewalIntervalInSecs": 10,
			"durationInSecs": 30,
			"registrationTimestamp": 1540538239969,
			"lastRenewalTimestamp": 1540781826379,
			"evictionTimestamp": 0,
			"serviceUpTimestamp": 1540538239463
		},
		"metadata": {
			"instanceId": "oneitom-dfss::dfss-service:50000",
			"iplatformtype": "平台服务",
			"weight": "1",
			"trackSampling": "0.1",
			"management.context-path": "/dfssservice"
		},
		"homePageUrl": "http://localhost:50000/dfssservice",
		"statusPageUrl": "http://localhost:50000/dfssservice/info",
		"healthCheckUrl": "http://localhost:50000/dfssservice/health",
		"secureHealthCheckUrl": "http://localhost:50000/dfssservice/health",
		"vipAddress": "dfss-service",
		"secureVipAddress": "dfss-service",
		"isCoordinatingDiscoveryServer": "false",
		"lastUpdatedTimestamp": "1540538239969",
		"lastDirtyTimestamp": "1540538239434",
		"actionType": "ADDED"
	}
}
```

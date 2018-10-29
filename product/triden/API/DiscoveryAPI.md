# 注册中心集成

> 作者 于胜强

## 注册应用实例（首次）

`curl -H "Content-Type:application/json" -X POST --data '{payload}'  http://localhost/eureka/apps/{appID}`


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

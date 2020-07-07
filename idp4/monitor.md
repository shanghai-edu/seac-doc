# 2.3.11 IdP 的监控

### 简介
要确保 IdP 服务的长期稳定运行，服务监控是必不可少的。除了常规的服务器监控之外，IdP 本身也暴露了一部分内置的监控指标，可供我们采集。

### status
访问状态监控页
```
curl http://127.0.0.1:8080/idp/profile/status
```
或者
```
curl http://127.0.0.1:8080/idp/status  
```
可以获取到 IdP 的状态监控数据
```
### Operating Environment Information
operating_system: Linux
operating_system_version: 3.10.0-1062.12.1.el7.x86_64
operating_system_architecture: amd64
jdk_version: 1.8.0_242
available_cores: 4
used_memory: 258 MB
maximum_memory: 843 MB

### Identity Provider Information
idp_version: 3.4.6
start_time: 2020-03-06T19:47:07+08:00
current_time: 2020-03-06T20:06:15+08:00
uptime: 1148872 ms

service: shibboleth.LoggingService
last successful reload attempt: 2020-03-06T11:45:46Z
last reload attempt: 2020-03-06T11:45:46Z

service: shibboleth.ReloadableAccessControlService
last successful reload attempt: 2020-03-06T11:45:49Z
last reload attempt: 2020-03-06T11:45:49Z

service: shibboleth.MetadataResolverService
last successful reload attempt: 2020-03-06T11:45:48Z
last reload attempt: 2020-03-06T11:45:48Z

        metadata source: HTTPMetadata
        last refresh attempt: 2020-03-06T12:00:54Z
        last successful refresh: 2020-03-06T12:00:54Z
        last update: 2020-03-06T11:45:54Z
        root validUntil: 2020-04-03T11:31:30Z

        metadata source: HTTPMetadataShec
        last refresh attempt: 2020-03-06T12:00:55Z
        last successful refresh: 2020-03-06T12:00:55Z
        last update: 2020-03-06T11:45:54Z

service: shibboleth.RelyingPartyResolverService
last successful reload attempt: 2020-03-06T11:45:48Z
last reload attempt: 2020-03-06T11:45:48Z

service: shibboleth.NameIdentifierGenerationService
last successful reload attempt: 2020-03-06T11:45:47Z
last reload attempt: 2020-03-06T11:45:47Z

service: shibboleth.AttributeResolverService
last successful reload attempt: 2020-03-06T11:45:47Z
last reload attempt: 2020-03-06T11:45:47Z

        DataConnector staticAttributes: has never failed

        DataConnector ComputedIDConnector: has never failed

        DataConnector myLDAP: has never failed

service: shibboleth.AttributeFilterService
last successful reload attempt: 2020-03-06T11:45:47Z
last reload attempt: 2020-03-06T11:45:47Z
```

### json status
在 IdP 3.3 之后的版本中，IdP 提供了 json 化的数据指标接口，从而更容易程序处理。
```
curl http://127.0.0.1:8080/idp/profile/admin/metrics
```
将返回 json 格式的监控数据，格式化后结构如下所示：
```
{
	"version": "3.0.0",
	"gauges": {
		"cores.available": {
			"value": 4
		},
		"host.name": {
			"value": "idp"
		},
		"java.class.path": {
			"value": "/usr/share/tomcat/bin/bootstrap.jar:/usr/share/tomcat/bin/tomcat-juli.jar:/usr/share/java/commons-daemon.jar"
		},
		"java.home": {
			"value": "/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre"
		},
		"java.vendor": {
			"value": "Oracle Corporation"
		},
		"java.vendor.url": {
			"value": "http://java.oracle.com/"
		},
		"java.version": {
			"value": "1.8.0_242"
		},
		"memory.free.bytes": {
			"value": 163839568
		},
		"memory.free.megs": {
			"value": 156
		},
		"memory.usage": {
			"value": 0.6362058486316756
		},
		"memory.used.bytes": {
			"value": 286523824
		},
		"memory.used.megs": {
			"value": 273
		},
		"net.shibboleth.idp.accesscontrol.reload.attempt": {
			"value": "2020-03-06T11:45:49.134Z"
		},
		"net.shibboleth.idp.accesscontrol.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.accesscontrol.reload.success": {
			"value": "2020-03-06T11:45:49.134Z"
		},
		"net.shibboleth.idp.attribute.filter.reload.attempt": {
			"value": "2020-03-06T11:45:47.443Z"
		},
		"net.shibboleth.idp.attribute.filter.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.attribute.filter.reload.success": {
			"value": "2020-03-06T11:45:47.443Z"
		},
		"net.shibboleth.idp.attribute.resolver.failure": {
			"value": {}
		},
		"net.shibboleth.idp.attribute.resolver.reload.attempt": {
			"value": "2020-03-06T11:45:47.517Z"
		},
		"net.shibboleth.idp.attribute.resolver.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.attribute.resolver.reload.success": {
			"value": "2020-03-06T11:45:47.517Z"
		},
		"net.shibboleth.idp.logging.reload.attempt": {
			"value": "2020-03-06T11:45:46.715Z"
		},
		"net.shibboleth.idp.logging.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.logging.reload.success": {
			"value": "2020-03-06T11:45:46.715Z"
		},
		"net.shibboleth.idp.metadata.refresh": {
			"value": {
				"HTTPMetadata": "2020-03-06T12:00:54.318Z",
				"HTTPMetadataShec": "2020-03-06T12:00:55.260Z"
			}
		},
		"net.shibboleth.idp.metadata.reload.attempt": {
			"value": "2020-03-06T11:45:48.248Z"
		},
		"net.shibboleth.idp.metadata.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.metadata.reload.success": {
			"value": "2020-03-06T11:45:48.248Z"
		},
		"net.shibboleth.idp.metadata.rootValidUntil": {
			"value": {
				"HTTPMetadata": "2020-04-03T11:31:30.000Z"
			}
		},
		"net.shibboleth.idp.metadata.successfulRefresh": {
			"value": {
				"HTTPMetadata": "2020-03-06T12:00:54.318Z",
				"HTTPMetadataShec": "2020-03-06T12:00:55.260Z"
			}
		},
		"net.shibboleth.idp.metadata.update": {
			"value": {
				"HTTPMetadata": "2020-03-06T11:45:54.028Z",
				"HTTPMetadataShec": "2020-03-06T11:45:54.093Z"
			}
		},
		"net.shibboleth.idp.nameid.reload.attempt": {
			"value": "2020-03-06T11:45:47.986Z"
		},
		"net.shibboleth.idp.nameid.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.nameid.reload.success": {
			"value": "2020-03-06T11:45:47.986Z"
		},
		"net.shibboleth.idp.relyingparty.reload.attempt": {
			"value": "2020-03-06T11:45:48.035Z"
		},
		"net.shibboleth.idp.relyingparty.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.relyingparty.reload.success": {
			"value": "2020-03-06T11:45:48.035Z"
		},
		"net.shibboleth.idp.starttime": {
			"value": "2020-03-06T11:45:45.798Z"
		},
		"net.shibboleth.idp.uptime": {
			"value": 1318624
		},
		"net.shibboleth.idp.version": {
			"value": "3.4.6"
		},
		"org.opensaml.version": {
			"value": "3.4.5"
		},
		"os.arch": {
			"value": "amd64"
		},
		"os.name": {
			"value": "Linux"
		},
		"os.version": {
			"value": "3.10.0-1062.12.1.el7.x86_64"
		}
	},
	"counters": {
		"net.shibboleth.idp.authn.ldap.successes": {
			"count": 37
		}
	},
	"histograms": {},
	"meters": {},
	"timers": {}
}
```

### 访问控制
默认情况下，idp 的状态监控接口仅允许本地访问，如果需要被其他地方访问的话，则需要增加 acl 设置。修改 `conf/access-control.xml` 配置文件，增加允许的 ip 地址
```
        <entry key="AccessByIPAddress">
            <bean id="AccessByIPAddress" parent="shibboleth.IPRangeAccessControl"
                p:allowedRanges="#{ {'127.0.0.1/32', '::1/128'} }" />
        </entry>
```
# IdP 的监控

### 简介
要确保 IdP 服务的长期稳定运行，服务监控是必不可少的。除了常规的服务器监控之外，IdP 本身也暴露了一部分内置的监控指标，可供我们采集。

### json status
在 IdP 4 已经取消了之前的状态页面，现在提供了 json 化的数据指标接口，从而更容易程序处理。
```
curl http://127.0.0.1:8080/idp/profile/admin/metrics
```
将返回 json 格式的监控数据，格式化后结构如下所示：
```
{
	"version": "4.0.0",
	"gauges": {
		"cores.available": {
			"value": 4
		},
		"host.name": {
			"value": "idp4"
		},
		"java.class.path": {
			"value": "/opt/tomcat/latest/bin/bootstrap.jar:/opt/tomcat/latest/bin/tomcat-juli.jar"
		},
		"java.home": {
			"value": "/usr/lib/jvm/java-11-openjdk-11.0.8.10-1.el7.x86_64"
		},
		"java.vendor": {
			"value": "N/A"
		},
		"java.vendor.url": {
			"value": "https://openjdk.java.net/"
		},
		"java.version": {
			"value": "11.0.8"
		},
		"memory.free.bytes": {
			"value": 884032864
		},
		"memory.free.megs": {
			"value": 843
		},
		"memory.usage": {
			"value": 0.356182178498771
		},
		"memory.used.bytes": {
			"value": 492217976
		},
		"memory.used.megs": {
			"value": 469
		},
		"net.shibboleth.idp.accesscontrol.reload.attempt": {
			"value": "2021-02-20T12:43:31.927098Z"
		},
		"net.shibboleth.idp.accesscontrol.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.accesscontrol.reload.success": {
			"value": "2021-02-20T12:43:31.927098Z"
		},
		"net.shibboleth.idp.attribute.filter.reload.attempt": {
			"value": "2021-02-20T12:43:31.267658Z"
		},
		"net.shibboleth.idp.attribute.filter.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.attribute.filter.reload.success": {
			"value": "2021-02-20T12:43:31.267658Z"
		},
		"net.shibboleth.idp.attribute.resolver.failure": {
			"value": {}
		},
		"net.shibboleth.idp.attribute.resolver.reload.attempt": {
			"value": "2021-02-20T12:43:31.362268Z"
		},
		"net.shibboleth.idp.attribute.resolver.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.attribute.resolver.reload.success": {
			"value": "2021-02-20T12:43:31.362268Z"
		},
		"net.shibboleth.idp.cas.registry.reload.attempt": {
			"value": "2021-02-20T12:43:31.949892Z"
		},
		"net.shibboleth.idp.cas.registry.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.cas.registry.reload.success": {
			"value": "2021-02-20T12:43:31.949892Z"
		},
		"net.shibboleth.idp.logging.reload.attempt": {
			"value": "2021-02-20T12:43:30.025333Z"
		},
		"net.shibboleth.idp.logging.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.logging.reload.success": {
			"value": "2021-02-20T12:43:30.025333Z"
		},
		"net.shibboleth.idp.managedbean.reload.attempt": {
			"value": "2021-02-20T12:43:31.967866Z"
		},
		"net.shibboleth.idp.managedbean.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.managedbean.reload.success": {
			"value": "2021-02-20T12:43:31.967866Z"
		},
		"net.shibboleth.idp.metadata.error": {
			"value": {}
		},
		"net.shibboleth.idp.metadata.refresh": {
			"value": {}
		},
		"net.shibboleth.idp.metadata.reload.attempt": {
			"value": "2021-02-20T12:43:31.835935Z"
		},
		"net.shibboleth.idp.metadata.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.metadata.reload.success": {
			"value": "2021-02-20T12:43:31.835935Z"
		},
		"net.shibboleth.idp.metadata.rootValidUntil": {
			"value": {}
		},
		"net.shibboleth.idp.metadata.successfulRefresh": {
			"value": {}
		},
		"net.shibboleth.idp.metadata.update": {
			"value": {}
		},
		"net.shibboleth.idp.nameid.reload.attempt": {
			"value": "2021-02-20T12:43:31.555423Z"
		},
		"net.shibboleth.idp.nameid.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.nameid.reload.success": {
			"value": "2021-02-20T12:43:31.555423Z"
		},
		"net.shibboleth.idp.relyingparty.reload.attempt": {
			"value": "2021-02-20T12:43:31.608783Z"
		},
		"net.shibboleth.idp.relyingparty.reload.error": {
			"value": null
		},
		"net.shibboleth.idp.relyingparty.reload.success": {
			"value": "2021-02-20T12:43:31.608783Z"
		},
		"net.shibboleth.idp.starttime": {
			"value": "2021-02-20T12:43:28.912Z"
		},
		"net.shibboleth.idp.uptime": {
			"value": 94286662
		},
		"net.shibboleth.idp.version": {
			"value": "4.0.1"
		},
		"org.opensaml.version": {
			"value": "4.0.1"
		},
		"os.arch": {
			"value": "amd64"
		},
		"os.name": {
			"value": "Linux"
		},
		"os.version": {
			"value": "3.10.0-957.el7.x86_64"
		}
	},
	"counters": {},
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
# 2.1.4 metadata 获取和注册

# 同步 metadata 配置

修改 ```IDP_HOME/conf/relying_party.xml``` 

metadata provider 修改,从联盟侧获取 metadata
```
        <metadata:MetadataProvider id="URLMD" xsi:type="metadata:FileBackedHTTPMetadataProvider"
                          metadataURL="https://ds.shec.edu.cn/metadata.xml"
                          backingFile="/opt/idp/metadata/allinone.xml">
        </metadata:MetadataProvider>
```

# 注册

idp 安装完毕启动无报错后,提交 `IDP_HOME/metadata/idp-metadata.xml` 到 its@cloud.sh.edu.cn 注册

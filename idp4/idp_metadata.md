# idp 注册

#### Metadata 同步
修改 `metadata/metadata-providers.xml` ，同步上海教育认证中心的 `metadata`
```xml
    <MetadataProvider id="HTTPMetadataShec"
                      xsi:type="FileBackedHTTPMetadataProvider"
                      backingFile="/opt/shibboleth-idp/metadata/ds.shec.edu.cn.xml"
                      minRefreshDelay="PT5M" 
                      maxRefreshDelay="PT10M" 
                      metadataURL="https://ds.shec.edu.cn/metadata.xml">
    </MetadataProvider>
```

#### 注册
修改 `metadata/idp-metadata.xml` 文件，在最后部分增加如下配置,以便于 metadata 合并的自动化处理

```xml
    </AttributeAuthorityDescriptor>
    <Organization xmlns="urn:oasis:names:tc:SAML:2.0:metadata">
        <OrganizationName xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xml:lang="en">seac</OrganizationName>
        <OrganizationDisplayName xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xml:lang="zh-CN">上海市教委</OrganizationDisplayName>
        <OrganizationURL xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xml:lang="en">https://idp.seac.edu.cn/</OrganizationURL>
    </Organization>
</EntityDescriptor>
```

然后邮件 its@cloud.sh.edu.cn 申请上线


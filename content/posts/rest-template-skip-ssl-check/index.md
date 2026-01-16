---
title: RestTemplate 跳过 HTTPS 检查
date: 2022-04-30 11:43:05+08:00
categories:
- Java
tags:
- java
- spring
- springboot
- https
- ssl
summary: 使用 RestTemplate 请求 https 的接口时，遇到关于证书的错误。
---

## 错误日志

```text
org.springframework.web.client.ResourceAccessException: I/O error on POST request for "https://ceshi.bczzhsq.com/api/v1/cmu/dahua/GetNewLists": sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; nested exception is javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target

 ......
Caused by: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
 at sun.security.provider.certpath.SunCertPathBuilder.build(SunCertPathBuilder.java:141)
 at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:126)
 at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:280)
 at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:434)
 ... 103 more
```

## 尝试使用 curl 访问

```bash
[root@localhost dahua]# curl -X POST https://ceshi.bczzhsq.com/api/v1/cmu/dahua/GetNewLists
curl: (60) Peer's Certificate issuer is not recognized.
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

也不行

## 跳过 https 证书检查

```java
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.web.client.RestTemplate;

import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import java.nio.charset.StandardCharsets;

/**
 * Web 相关配置
 *
 * @author WZW  2022-04-23 10:15
 */
@Configuration
public class WebConfig {

    /**
     * 构造 RestTemplate
     *
     * @return RestTemplate
     * @throws Exception
     */
    @Bean
    public RestTemplate restTemplate() throws Exception {
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
        // 超时
        factory.setConnectionRequestTimeout(5000);
        factory.setConnectTimeout(5000);
        factory.setReadTimeout(5000);
        SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(createIgnoreVerifySSL(),
                // 指定 TLS 版本
                null,
                // 指定算法
                null,
                // 取消域名验证
                (string, ssls) -> true);
        CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf).build();
        factory.setHttpClient(httpClient);
        RestTemplate restTemplate = new RestTemplate(factory);
        // 解决中文乱码问题
        restTemplate.getMessageConverters().set(1, new StringHttpMessageConverter(StandardCharsets.UTF_8));
        return restTemplate;
    }

    /**
     * 跳过证书效验的 SSLContext
     *
     * @return SSLContext
     * @throws Exception
     */
    private static SSLContext createIgnoreVerifySSL() throws Exception {
        SSLContext sc = SSLContext.getInstance("TLS");

        // 实现一个 X509TrustManager 接口，用于绕过验证，不用修改里面的方法
        X509TrustManager trustManager = new X509TrustManager() {
            @Override
            public void checkClientTrusted(java.security.cert.X509Certificate[] paramArrayOfX509Certificate,
                                           String paramString) {
            }

            @Override
            public void checkServerTrusted(java.security.cert.X509Certificate[] paramArrayOfX509Certificate,
                                           String paramString) {
            }

            @Override
            public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                return null;
            }
        };
        sc.init(null, new TrustManager[]{trustManager}, null);
        return sc;
    }
}
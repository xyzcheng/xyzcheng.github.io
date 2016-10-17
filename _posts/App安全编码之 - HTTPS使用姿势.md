---
layout: post
title:  "App安全编码之 - HTTPS使用姿势"
date:   2015-03-26 15:14:54
categories: Sublime
tags: Sublime jshint csslint
---

* content
{:toc}

App安全编码之 - HTTPS使用姿势
乜聚虎2月26日
通用代码设计

很多app需要与服务器通信，传输一些关乎用户财产安全的敏感数据，这时候就需要使用https协议保证数据的安全。然而有时开发者在编写https相关代码时处理不当，结果适得其反，不但没有起到保护作用，反而加重了系统的负担，赔妇折兵。本文不讨论https协议本身的安全性，只是列举几个常见的https使用上的问题，包含错误与正确的编码方式，如有表述不当之处欢迎提出讨论。关于如何正确使用https的更详细说明，请参考开发者网站，本文大部分内容引用于此，后面具体引用的部分不再说明。
Https编码风险分析

App使用SDK与https网站建立连接时的安全校验，可简单描述为两个步骤： 首先校验网站的证书是否由受信任的根证书签发（可以有中间证书；Android系统预置了大量被广泛信任的根证书，可以满足大部分需求）； 然后校验证书的名称（subject/CN)是否与网站域名相匹配。两者都通过才继续，否则抛出对应的异常。

一般https网站的证书由受信任的根证书签发，并且名称与域名相匹配，与这些网站通信时app并不需要特殊处理，即可保证数据的安全。但是并非所有网站都如此配置，两种https网站配置如下：

    网站证书不是由受信任的根证书签发。这些证书可能是： 自签发的证书、私有的根证书签发的证书，或者根证书虽然可信，但是Android并不支持。

    证书名称与网站域名不匹配

很多app为了能正常与这些https网站通信，采用了简单粗暴的错误处理方式，导致https几乎完全失效，下面一一列举。

为了方便测试，我创建了3个https网站，证书配置分别如下。

    test.andrsec.com 使用Android信任的根证书（Certification Authority of WoSign）签发的证书，名称（subject）为test.andrsec.com

    test1.andrsec.com 使用自签名的证书，名称为test1.andrsec.com

    test2.andrsec.com 和test.andrsec.com共用一个证书，如前所说，名称为test.andrsec.com

错误处理1：信任任何证书

一些https网站使用了不受信任的证书，例如我的test1.andrsec.com使用了自签名的证书，默认情况下App在与这些网站通信时会抛出证书不可信的异常。

javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.


为了处理异常，有些App简单粗暴的采用了信任所有证书的策略，代码如下。

// Create a TrustManager which trust any CAs
    TrustManager tm = new X509TrustManager() {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            // Do nothing
        }
        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
            // Do nothing
        }
    };

    try {
        // Using the foolish TrustManager.
        TrustManager[] tms = new TrustManager[]{tm};
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null, tms, null);
        URL url = new URL(SELFSIGN_URL); // test1.andrsec.com
        HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
        conn.setSSLSocketFactory(sslContext.getSocketFactory());
        InputStream in = conn.getInputStream();
        output(in);
    } catch (Exception e) {
        output(e);
    }

使用抓包工具，以及任意证书，即可伪装https网站，轻松获取传输的内容，篡改也仅仅是几行代码的事情。

testss.png
testss.png

正确的处理方式是，仅将该网站所使用的证书作为信任的证书，而不是信任所有。以自签名的证书为例，参考代码如下。

String SELF_CA = "-----BEGIN CERTIFICATE-----" + ... // CA in PEM format
        
        // Load CAs from an ByteArrayStream
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        // SELF_CA is the certificate for "test1.andrsec.com"
        InputStream caInput = new BufferedInputStream(new ByteArrayInputStream(SELF_CA.getBytes()));
        Certificate ca;
        ca = cf.generateCertificate(caInput);

        // Create a KeyStore containing our trusted CAs
        String keyStoreType = KeyStore.getDefaultType();
        KeyStore keyStore = KeyStore.getInstance(keyStoreType);
        keyStore.load(null, null);
        keyStore.setCertificateEntry("ca", ca);

        // Create a TrustManager that trusts the CAs in our KeyStore
        String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
        TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
        tmf.init(keyStore);

        // Create an SSLContext that uses our TrustManager
        SSLContext context = SSLContext.getInstance("TLS");
        context.init(null, tmf.getTrustManagers(), null);

        // Tell the URLConnection to use a SocketFactory from our SSLContext
        URL url = new URL(SELFSIGN_URL); // "https://test1.andrsec.com/index.html"
        HttpsURLConnection urlConnection =
                (HttpsURLConnection) url.openConnection();
        urlConnection.setSSLSocketFactory(context.getSocketFactory());
        InputStream in = urlConnection.getInputStream();
        output(in);

错误处理2：不校验Hostname是否与证书匹配

一般情况下，https网站证书的名称应该与该网站的域名相匹配，例如google.com的证书名称为*.google.com。极少情况下，两者不匹配，例如我的test2.andrsec.com使用了受信任的证书，但是名称却是test.andrsec.com。这种情况App会抛出域名校验异常（Android 2.2之前virtual hosting也会导致这个问题）。

java.io.IOException: Hostname 'xxx.com' was not verified

同样有些App简单粗暴的采用了不校验域名的策略，代码如下。

    错误代码1

    // Create an empty HostnameVerifier witch trust any hostname.
      HostnameVerifier hostnameVerifier = new HostnameVerifier() {
          @Override
          public boolean verify(String hostname, SSLSession session) {
              // Always return true.
              return true;
          }
      };

      // Tell the URLConnection to use our HostnameVerifier
      try {
          URL url = new URL(HOSTNAME_URL); // test2.andrsec.com
          HttpsURLConnection urlConnection =
                  (HttpsURLConnection) url.openConnection();
          urlConnection.setHostnameVerifier(hostnameVerifier);
          InputStream in = urlConnection.getInputStream();
          output(in);
      } catch (Exception e) {
          output(e);
      }

    错误代码2

    URL url = new URL(HOSTNAME_URL); // test2.andrsec.com
      HttpsURLConnection urlConnection =
          (HttpsURLConnection)url.openConnection();
      // All all hostname
      urlConnection.setHostnameVerifier(new AllowAllHostnameVerifier());
      InputStream in = urlConnection.getInputStream();
      output(in)

使用抓包工具，以及任意一个由受信任的根证书签发的证书，即可伪装https网站，获取传输的内容，篡改也仅仅是几行代码的事情。

hostname.png
hostname.png

正确的处理办法需要具体问题具体分析。可能大部分情况下这种错误只是开发者的画蛇添足，所以首先还是检查一下证书的名称是否真的有问题。如果真出现了域名与证书名称不匹配的情况，如我的test2.andrsec.com，可以使用我的test.andrsec.com域名来与证书进行匹配校验。这种处理方法的前提是，两个域名和证书属于同一个人。

// Create an HostnameVerifier that hardwires the expected hostname.
    // Note that is different than the URL's hostname:
    // test.andrsec.com versus test2.andrsec.com
    HostnameVerifier hostnameVerifier = new HostnameVerifier() {
        @Override
        public boolean verify(String hostname, SSLSession session) {
            HostnameVerifier hv =
                    HttpsURLConnection.getDefaultHostnameVerifier();
            return hv.verify("test.andrsec.com", session);
        }
    };

    // Tell the URLConnection to use our HostnameVerifier
    try {
        URL url = new URL(HOSTNAME_URL); // test2.andrsec.com
        HttpsURLConnection urlConnection =
                (HttpsURLConnection) url.openConnection();
        urlConnection.setHostnameVerifier(hostnameVerifier);
        InputStream in = urlConnection.getInputStream();
        output(in);
    } catch (Exception e) {
        output(e);
    }

风险规避

    如前所述，关于如何正确使用https，参考开发者网站。

    一般App只与固定的一个或几个https网站通信，这种情况推荐使用所谓的pinning技术，即只信任与App通信的Https网站的证书（和/或其根证书）。防止由于系统中其他根证书被攻击而带来各种安全问题。

总结

    不要信任所有证书

    不要忽略域名校验

    尽可能采用pinning： 只信任自己https网站的证书（根证书）

    赶快检查一下你的app: JIRA

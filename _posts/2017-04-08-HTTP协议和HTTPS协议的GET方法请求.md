---
layout: post
title: HTTP 协议和 HTTPS 协议的 GET 方法请求
category : [问题记录]
tagline: "Supporting tagline"
tags : [HTTP, HTTPS]
---
{% include JB/setup %}
# HTTP 协议和 HTTPS 协议的 GET 方法请求
---

## HTTP 协议请求

#### 方法一：

获取 HTTP 协议请求状态码：

```
public int requestHttpGet(String url) {
    HttpGet httpGet = new HttpGet(url);
    RequestConfig requestConfig = RequestConfig.custom()
                        .setConnectTimeout(9000).setConnectionRequestTimeout(9000).build();
    httpGet.setConfig(requestConfig);
    CloseableHttpClient httpClient = null;
    HttpResponse response;
    try {
        httpClient = HttpClients.createDefault();
        response = httpClient.execute(httpGet);
        return response.getStatusLine().getStatusCode();
    } catch (Exception e) {
        log.error("HTTP 协议 GET 请求出错：" + e.getMessage());
        return 408;
    } finally {
        try {
            httpClient.close();
        } catch (IOException e) {
            log.error("关闭 CloseableHttpClient 对象出现错误：" + e.getMessage());
        }
    }
}
```

<!--break-->

#### 方法二：

```
public int requestHttpGet(String getUrl) {
    URL url = new URL(getUrl);
    java.net.HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    return conn.getResponseCode();
}
```


## HTTPS 协议请求（无法请求自签名的 HTTPS 协议）

获取 HTTPS 协议请求状态码：

```
    public CloseableHttpClient createSSLHttpClient() {   
        char SEP = File.separatorChar;
        final SSLConnectionSocketFactory sslsf;
        try {
            sslsf = new SSLConnectionSocketFactory(SSLContext.getDefault(),
                    NoopHostnameVerifier.INSTANCE);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
        final Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory>create()
                .register("http", new PlainConnectionSocketFactory())
                .register("https", sslsf)
                .build();
        final PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(registry);
        cm.setMaxTotal(100);
        return HttpClients.custom()
                .setSSLSocketFactory(sslsf)
                .setConnectionManager(cm)
                .build();
    }
    
    
    public int requestHttpGet(String url, RequestConfig requestConfig) {
        HttpGet httpGet = new HttpGet(url);
        httpGet.setConfig(requestConfig);
        CloseableHttpClient httpClient = null;
        HttpResponse response = null;
        try {
            httpClient = createSSLHttpClient();
            response = httpClient.execute(httpGet);
            return response.getStatusLine().getStatusCode();
        } catch (Exception e) {
            e.printStackTrace();
            return 408;
        } finally {
            try {
                httpClient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
        
        
    public int getStatusCode() {
        RequestConfig requestConfig = RequestConfig.custom()
                            .setConnectTimeout(9000).setConnectionRequestTimeout(9000).build();
        return requestHttpsGet(url, requestConfig);
    }
```

这个方法同时也可以获取 HTTP 协议请求的状态码。


## HTTPS 协议请求（可以请求自签名的 HTTPS 协议）

> [[keyStore 和 truststore 区别](http://blog.csdn.net/chenzanlong123/article/details/11784143)

#### 方法一

> 参考：
> https://hc.apache.org/httpclient-3.x/sslguide.html
> [通过 HTTPClient 访问启用 SSL 的 Quickr REST API](http://chnwaterloo.iteye.com/blog/690549)

EasyX509TrustManager.java

```
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import javax.net.ssl.TrustManager;
import javax.net.ssl.TrustManagerFactory;
import javax.net.ssl.X509TrustManager;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;


public class EasyX509TrustManager implements X509TrustManager {

    private X509TrustManager standardTrustManager = null;

    /**
     * Log object for this class.
     */
    private static final Log LOG = LogFactory.getLog(EasyX509TrustManager.class);

    /**
     * Constructor for EasyX509TrustManager.
     */
    public EasyX509TrustManager(KeyStore keystore) throws NoSuchAlgorithmException, KeyStoreException {
        super();
        TrustManagerFactory factory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        factory.init(keystore);
        TrustManager[] trustmanagers = factory.getTrustManagers();
        if (trustmanagers.length == 0) {
            throw new NoSuchAlgorithmException("no trust manager found");
        }
        this.standardTrustManager = (X509TrustManager) trustmanagers[0];
    }

    /**
     * @see X509TrustManager#checkClientTrusted(X509Certificate[], String authType)
     */
    public void checkClientTrusted(X509Certificate[] certificates, String authType) throws CertificateException {
        standardTrustManager.checkClientTrusted(certificates, authType);
    }

    /**
     * @see X509TrustManager#checkServerTrusted(X509Certificate[], String authType)
     */
    public void checkServerTrusted(X509Certificate[] certificates, String authType) throws CertificateException {
//        if ((certificates != null) && LOG.isDebugEnabled()) {
//            LOG.debug("Server certificate chain:");
//            for (int i = 0; i < certificates.length; i++) {
//                LOG.debug("X509Certificate[" + i + "]=" + certificates[i]);
//            }
//        }
//        if ((certificates != null) && (certificates.length == 1)) {
//            certificates[0].checkValidity();
//        } else {
//            standardTrustManager.checkServerTrusted(certificates,authType);
//        }
    }

    /**
     * @see X509TrustManager#getAcceptedIssuers()
     */
    public X509Certificate[] getAcceptedIssuers() {
        return this.standardTrustManager.getAcceptedIssuers();
    }

}
```

EasySSLProtocolSocketFactory.java

```
import java.io.IOException;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.Socket;
import java.net.SocketAddress;
import java.net.UnknownHostException;

import javax.net.SocketFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;

import org.apache.commons.httpclient.ConnectTimeoutException;
import org.apache.commons.httpclient.HttpClientError;
import org.apache.commons.httpclient.params.HttpConnectionParams;
import org.apache.commons.httpclient.protocol.ProtocolSocketFactory;
import org.apache.commons.httpclient.protocol.SecureProtocolSocketFactory;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;


public class EasySSLProtocolSocketFactory implements ProtocolSocketFactory {

    /**
     * Log object for this class.
     */
    private static final Log LOG = LogFactory.getLog(EasySSLProtocolSocketFactory.class);

    private SSLContext sslcontext = null;

    /**
     * Constructor for EasySSLProtocolSocketFactory.
     */
    public EasySSLProtocolSocketFactory() {
        super();
    }

    private static SSLContext createEasySSLContext() {
        try {
            SSLContext context = SSLContext.getInstance("SSL");
            context.init(
                    null,
                    new TrustManager[]{new EasyX509TrustManager(null)},
                    null);
            return context;
        } catch (Exception e) {
            LOG.error(e.getMessage(), e);
            throw new HttpClientError(e.toString());
        }
    }

    private SSLContext getSSLContext() {
        if (this.sslcontext == null) {
            this.sslcontext = createEasySSLContext();
        }
        return this.sslcontext;
    }

    /**
     * @see SecureProtocolSocketFactory#createSocket(String, int, InetAddress, int)
     */
    public Socket createSocket(
            String host,
            int port,
            InetAddress clientHost,
            int clientPort)
            throws IOException, UnknownHostException {

        return getSSLContext().getSocketFactory().createSocket(
                host,
                port,
                clientHost,
                clientPort
        );
    }

    /**
     * Attempts to get a new socket connection to the given host within the given time limit.
     * <p>
     * To circumvent the limitations of older JREs that do not support connect timeout a
     * controller thread is executed. The controller thread attempts to create a new socket
     * within the given limit of time. If socket constructor does not return until the
     * timeout expires, the controller terminates and throws an {@link ConnectTimeoutException}
     * </p>
     *
     * @param host       the host name/IP
     * @param port       the port on the host
     * @param clientHost the local host name/IP to bind the socket to
     * @param clientPort the port on the local machine
     * @param params     {@link HttpConnectionParams Http connection parameters}
     * @return Socket a new socket
     * @throws IOException          if an I/O error occurs while creating the socket
     * @throws UnknownHostException if the IP address of the host cannot be
     *                              determined
     */
    public Socket createSocket(
            final String host,
            final int port,
            final InetAddress localAddress,
            final int localPort,
            final HttpConnectionParams params
    ) throws IOException, UnknownHostException, ConnectTimeoutException {
        if (params == null) {
            throw new IllegalArgumentException("Parameters may not be null");
        }
        int timeout = params.getConnectionTimeout();
        SocketFactory socketfactory = getSSLContext().getSocketFactory();
        if (timeout == 0) {
            return socketfactory.createSocket(host, port, localAddress, localPort);
        } else {
            Socket socket = socketfactory.createSocket();
            SocketAddress localaddr = new InetSocketAddress(localAddress, localPort);
            SocketAddress remoteaddr = new InetSocketAddress(host, port);
            socket.bind(localaddr);
            socket.connect(remoteaddr, timeout);
            return socket;
        }
    }

    /**
     * @see SecureProtocolSocketFactory#createSocket(String, int)
     */
    public Socket createSocket(String host, int port)
            throws IOException, UnknownHostException {
        return getSSLContext().getSocketFactory().createSocket(
                host,
                port
        );
    }

    /**
     * @see SecureProtocolSocketFactory#createSocket(Socket, String, int, boolean)
     */
    public Socket createSocket(
            Socket socket,
            String host,
            int port,
            boolean autoClose)
            throws IOException, UnknownHostException {
        return getSSLContext().getSocketFactory().createSocket(
                socket,
                host,
                port,
                autoClose
        );
    }

    public boolean equals(Object obj) {
        return ((obj != null) && obj.getClass().equals(EasySSLProtocolSocketFactory.class));
    }

    public int hashCode() {
        return EasySSLProtocolSocketFactory.class.hashCode();
    }


}
```

获取 HTTPS 协议请求状态码：

需要导入以下依赖：

```
compile('org.apache.httpcomponents:httpclient')
compile('commons-httpclient:commons-httpclient:3.1')
```

```
public int requestHttpsGet(String url) {
    EasySSLProtocolSocketFactory easySSL;
    GetMethod httpGet = null;
    try {
        easySSL = new EasySSLProtocolSocketFactory();

        Protocol easyHttps = new Protocol("https", easySSL, 443);
        Protocol.registerProtocol("https", easyHttps);

        HttpClient httpClient = new HttpClient();
        httpClient.getHttpConnectionManager().getParams().setConnectionTimeout(9000);
        httpClient.getHttpConnectionManager().getParams().setSoTimeout(9000);
        httpClient.getParams().setContentCharset("UTF-8");

        httpGet = new GetMethod(url);
        long startTime = System.currentTimeMillis();
        httpClient.executeMethod(httpGet);
        long endTime = System.currentTimeMillis();
        time = endTime - startTime;
        return httpGet.getStatusCode();
    } catch (Exception e) {
        log.error("HTTPS 协议 GET 请求出错：" + e.getMessage());
        time = 9000;
        return 408;
    } finally {
        httpGet.releaseConnection();
    }
}

/**
 * 获取URL链接的Scheme，返回https或者http
 *
 * @param url
 * @return
 */
public static String getUrlScheme(String url) {
    String scheme = "";
    try {
        scheme = new URL(url).toURI().getScheme();
    } catch (URISyntaxException e) {
        log.error("处理url出现错误：" + e.getMessage());
    } catch (MalformedURLException e) {
        log.error("处理url出现错误：" + e.getMessage());
    }
    return scheme;
}
    
public int getStatusCode() {
    return requestHttpsGet(url);
}
```

#### 方法二

> 参考：[ java 访问 https 忽略证书](http://blog.csdn.net/xiyushiyi/article/details/46685387)

```
/**
  *import javax.net.ssl.*;
  *import java.net.HttpURLConnection;
  *import java.net.URL;
  *import java.security.cert.X509Certificate;
*/

final static HostnameVerifier DO_NOT_VERIFY = new HostnameVerifier() {

    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
};

public int requestHttpsGet(String getUrl) {
    HttpURLConnection conn;
    try {
        // Create a trust manager that does not validate certificate chains
        trustAllHosts();

        URL url = new URL(getUrl);

        HttpsURLConnection https = (HttpsURLConnection) url.openConnection();
        if (url.getProtocol().toLowerCase().equals("https")) {
            https.setHostnameVerifier(DO_NOT_VERIFY);
            conn = https;
            conn.connect();
            return conn.getResponseCode();
        }
    } catch (Exception e) {
        return 408;
    }
}

/**
 * Trust every server - don't check for any certificate
*/
private static void trustAllHosts() {

    // Create a trust manager that does not validate certificate chains
    TrustManager[] trustAllCerts = new TrustManager[]{new X509TrustManager() {

        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
            return new java.security.cert.X509Certificate[]{};
        }

        public void checkClientTrusted(X509Certificate[] chain, String authType) {

        }

        public void checkServerTrusted(X509Certificate[] chain, String authType) {

        }
    }};

    // Install the all-trusting trust manager
    try {
        SSLContext sc = SSLContext.getInstance("TLS");
        sc.init(null, trustAllCerts, new java.security.SecureRandom());
        HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

#### 方法三

> 参考：[SunCertPathBuilderException: unable to find valid certification path to requested target](https://www.mkyong.com/webservices/jax-ws/suncertpathbuilderexception-unable-to-find-valid-certification-path-to-requested-target/)

InstallCert.java（网上下载的源代码）

```
package com.aw.ad.util;
/*
 * Copyright 2006 Sun Microsystems, Inc.  All Rights Reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 *   - Redistributions of source code must retain the above copyright
 *     notice, this list of conditions and the following disclaimer.
 *
 *   - Redistributions in binary form must reproduce the above copyright
 *     notice, this list of conditions and the following disclaimer in the
 *     documentation and/or other materials provided with the distribution.
 *
 *   - Neither the name of Sun Microsystems nor the names of its
 *     contributors may be used to endorse or promote products derived
 *     from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
 * IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
 * PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
 * LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */
/**
 * http://blogs.sun.com/andreas/resource/InstallCert.java
 * Use:
 * java InstallCert hostname
 * Example:
 *% java InstallCert ecc.fedora.redhat.com
 */

import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;
import java.security.MessageDigest;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * Class used to add the server's certificate to the KeyStore
 * with your trusted certificates.
 */
public class InstallCert {

    public static void main(String[] args) throws Exception {
        String host;
        int port;
        char[] passphrase;
        if ((args.length == 1) || (args.length == 2)) {
            String[] c = args[0].split(":");
            host = c[0];
            port = (c.length == 1) ? 443 : Integer.parseInt(c[1]);
            String p = (args.length == 1) ? "changeit" : args[1];
            passphrase = p.toCharArray();
        } else {
            System.out.println("Usage: java InstallCert <host>[:port] [passphrase]");
            return;
        }

        File file = new File("jssecacerts");
        if (file.isFile() == false) {
            char SEP = File.separatorChar;
            File dir = new File(System.getProperty("java.home") + SEP
                    + "lib" + SEP + "security");
            file = new File(dir, "jssecacerts");
            if (file.isFile() == false) {
                file = new File(dir, "cacerts");
            }
        }
        System.out.println("Loading KeyStore " + file + "...");
        InputStream in = new FileInputStream(file);
        KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
        ks.load(in, passphrase);
        in.close();

        SSLContext context = SSLContext.getInstance("TLS");
        TrustManagerFactory tmf =
                TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(ks);
        X509TrustManager defaultTrustManager = (X509TrustManager) tmf.getTrustManagers()[0];
        SavingTrustManager tm = new SavingTrustManager(defaultTrustManager);
        context.init(null, new TrustManager[]{tm}, null);
        SSLSocketFactory factory = context.getSocketFactory();

        System.out.println("Opening connection to " + host + ":" + port + "...");
        SSLSocket socket = (SSLSocket) factory.createSocket(host, port);
        socket.setSoTimeout(10000);
        try {
            System.out.println("Starting SSL handshake...");
            socket.startHandshake();
            socket.close();
            System.out.println();
            System.out.println("No errors, certificate is already trusted");
        } catch (SSLException e) {
            System.out.println();
            e.printStackTrace(System.out);
        }

        X509Certificate[] chain = tm.chain;
        if (chain == null) {
            System.out.println("Could not obtain server certificate chain");
            return;
        }

        BufferedReader reader =
                new BufferedReader(new InputStreamReader(System.in));

        System.out.println();
        System.out.println("Server sent " + chain.length + " certificate(s):");
        System.out.println();
        MessageDigest sha1 = MessageDigest.getInstance("SHA1");
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        for (int i = 0; i < chain.length; i++) {
            X509Certificate cert = chain[i];
            System.out.println
                    (" " + (i + 1) + " Subject " + cert.getSubjectDN());
            System.out.println("   Issuer  " + cert.getIssuerDN());
            sha1.update(cert.getEncoded());
            System.out.println("   sha1    " + toHexString(sha1.digest()));
            md5.update(cert.getEncoded());
            System.out.println("   md5     " + toHexString(md5.digest()));
            System.out.println();
        }

        System.out.println("Enter certificate to add to trusted keystore or 'q' to quit: [1]");
        String line = reader.readLine().trim();
        int k;
        try {
            k = (line.length() == 0) ? 0 : Integer.parseInt(line) - 1;
        } catch (NumberFormatException e) {
            System.out.println("KeyStore not changed");
            return;
        }

        X509Certificate cert = chain[k];
        String alias = host + "-" + (k + 1);
        ks.setCertificateEntry(alias, cert);

        OutputStream out = new FileOutputStream("jssecacerts");
        ks.store(out, passphrase);
        out.close();

        System.out.println();
        System.out.println(cert);
        System.out.println();
        System.out.println
                ("Added certificate to keystore 'jssecacerts' using alias '"
                        + alias + "'");
    }

    private static final char[] HEXDIGITS = "0123456789abcdef".toCharArray();

    private static String toHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 3);
        for (int b : bytes) {
            b &= 0xff;
            sb.append(HEXDIGITS[b >> 4]);
            sb.append(HEXDIGITS[b & 15]);
            sb.append(' ');
        }
        return sb.toString();
    }

    private static class SavingTrustManager implements X509TrustManager {

        private final X509TrustManager tm;
        private X509Certificate[] chain;

        SavingTrustManager(X509TrustManager tm) {
            this.tm = tm;
        }

        public X509Certificate[] getAcceptedIssuers() {
            throw new UnsupportedOperationException();
        }

        public void checkClientTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
            throw new UnsupportedOperationException();
        }

        public void checkServerTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
            this.chain = chain;
            tm.checkServerTrusted(chain, authType);
        }
    }

}
```

InstallCert.java（稍加修改后的代码）

```
/*
 * Copyright 2006 Sun Microsystems, Inc.  All Rights Reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 *   - Redistributions of source code must retain the above copyright
 *     notice, this list of conditions and the following disclaimer.
 *
 *   - Redistributions in binary form must reproduce the above copyright
 *     notice, this list of conditions and the following disclaimer in the
 *     documentation and/or other materials provided with the distribution.
 *
 *   - Neither the name of Sun Microsystems nor the names of its
 *     contributors may be used to endorse or promote products derived
 *     from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
 * IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
 * PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
 * LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */
/**
 * http://blogs.sun.com/andreas/resource/InstallCert.java
 * Use:
 * java InstallCert hostname
 * Example:
 * % java InstallCert ecc.fedora.redhat.com
 */

import javax.net.ssl.*;
import java.io.*;
import java.security.KeyStore;
import java.security.MessageDigest;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

/**
 * Class used to add the server's certificate to the KeyStore
 * with your trusted certificates.
 */
public class InstallCert {

    public synchronized static void installCert(String url) throws Exception {
        String ipAddress = HttpUtils.convertUrlToIP(url);
        String[] args = ipAddress.split(":", 2);
        String host;
        int port;
        if ((args.length == 1) || (args.length == 2)) {
            host = args[0];
            port = args.length == 1 ? 443 : Integer.parseInt(args[1]);
        } else {
            System.out.println("Usage: java InstallCert <host>[:port] [passphrase]");
            return;
        }
        char[] passphrase = "changeit".toCharArray();
//        if ((args.length == 1) || (args.length == 2)) {
//            String[] c = args[0].split(":");
//            host = c[0];
//            port = (c.length == 1) ? 443 : Integer.parseInt(c[1]);
//            String p = (args.length == 1) ? "changeit" : args[1];
//            passphrase = p.toCharArray();
//        }
//        else {
//            System.out.println("Usage: java InstallCert <host>[:port] [passphrase]");
//            return;
//        }

        File file = new File("jssecacerts");
        if (file.isFile() == false) {
            char SEP = File.separatorChar;
            File dir = new File(System.getProperty("java.home") + SEP
                    + "lib" + SEP + "security");
            file = new File(dir, "jssecacerts");
            if (file.isFile() == false) {
                file = new File(dir, "cacerts");
            }
        }
        System.out.println("Loading KeyStore " + file + "...");
        InputStream in = new FileInputStream(file);
        KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
        ks.load(in, passphrase);
        in.close();

        SSLContext context = SSLContext.getInstance("TLS");
        TrustManagerFactory tmf =
                TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(ks);
        X509TrustManager defaultTrustManager = (X509TrustManager) tmf.getTrustManagers()[0];
        SavingTrustManager tm = new SavingTrustManager(defaultTrustManager);
        context.init(null, new TrustManager[]{tm}, null);
        SSLSocketFactory factory = context.getSocketFactory();

        System.out.println("Opening connection to " + host + ":" + port + "...");
        SSLSocket socket = (SSLSocket) factory.createSocket(host, port);
        socket.setSoTimeout(10000);
        try {
            System.out.println("Starting SSL handshake...");
            socket.startHandshake();
            socket.close();
            System.out.println();
            System.out.println("No errors, certificate is already trusted");
        } catch (SSLException e) {
            System.out.println();
            e.printStackTrace(System.out);
        }

        X509Certificate[] chain = tm.chain;
        if (chain == null) {
            System.out.println("Could not obtain server certificate chain");
            return;
        }

        BufferedReader reader =
                new BufferedReader(new InputStreamReader(System.in));

        System.out.println();
        System.out.println("Server sent " + chain.length + " certificate(s):");
        System.out.println();
        MessageDigest sha1 = MessageDigest.getInstance("SHA1");
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        for (int i = 0; i < chain.length; i++) {
            X509Certificate cert = chain[i];
            System.out.println
                    (" " + (i + 1) + " Subject " + cert.getSubjectDN());
            System.out.println("   Issuer  " + cert.getIssuerDN());
            sha1.update(cert.getEncoded());
            System.out.println("   sha1    " + toHexString(sha1.digest()));
            md5.update(cert.getEncoded());
            System.out.println("   md5     " + toHexString(md5.digest()));
            System.out.println();
        }

//        System.out.println("Enter certificate to add to trusted keystore or 'q' to quit: [1]");
//        String line = reader.readLine().trim();
//        int k;
//        try {
//            k = (line.length() == 0) ? 0 : Integer.parseInt(line) - 1;
//        } catch (NumberFormatException e) {
//            System.out.println("KeyStore not changed");
//            return;
//        }
        int k = 0;
        X509Certificate cert = chain[k];
        String alias = host + "-" + (k + 1);
        ks.setCertificateEntry(alias, cert);

        OutputStream out = new FileOutputStream("jssecacerts");
        ks.store(out, passphrase);
        out.close();

        System.out.println();
        System.out.println(cert);
        System.out.println();
        System.out.println
                ("Added certificate to keystore 'jssecacerts' using alias '"
                        + alias + "'");

//        char SEP = File.separatorChar;
//        FileUtils.copyFile(file.getAbsoluteFile().getPath(), System.getProperty("java.home") + SEP + "lib" + SEP + "security");
    }

    private static final char[] HEXDIGITS = "0123456789abcdef".toCharArray();

    private static String toHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 3);
        for (int b : bytes) {
            b &= 0xff;
            sb.append(HEXDIGITS[b >> 4]);
            sb.append(HEXDIGITS[b & 15]);
            sb.append(' ');
        }
        return sb.toString();
    }

    private static class SavingTrustManager implements X509TrustManager {

        private final X509TrustManager tm;
        private X509Certificate[] chain;

        SavingTrustManager(X509TrustManager tm) {
            this.tm = tm;
        }

        public X509Certificate[] getAcceptedIssuers() {
            throw new UnsupportedOperationException();
        }

        public void checkClientTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
            throw new UnsupportedOperationException();
        }

        public void checkServerTrusted(X509Certificate[] chain, String authType)
                throws CertificateException {
            this.chain = chain;
            tm.checkServerTrusted(chain, authType);
        }
    }

}
```

获取 HTTPS 协议请求状态码：

```
public CloseableHttpClient createSSLHttpClient() {
    char SEP = File.separatorChar;
    System.setProperty("javax.net.ssl.trustStore", System.getProperty("user.dir") + SEP + "jssecacerts");
    final SSLConnectionSocketFactory sslsf;
    try {
        sslsf = new SSLConnectionSocketFactory(SSLContext.getDefault(),
                NoopHostnameVerifier.INSTANCE);
    } catch (NoSuchAlgorithmException e) {
        throw new RuntimeException(e);
    }
    final Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory>create()
            .register("http", new PlainConnectionSocketFactory())
            .register("https", sslsf)
            .build();
    final PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager(registry);
    cm.setMaxTotal(100);
    return HttpClients.custom()
            .setSSLSocketFactory(sslsf)
            .setConnectionManager(cm)
            .build();
}


public synchronized int requestHttpsGet(String url) {
    HttpGet httpGet = new HttpGet(url);
    httpGet.setConfig(requestConfig);
    HttpResponse response = null;
    CloseableHttpClient httpClient = null;
    InstallCert.installCert(url);
    try {
        httpClient = createSSLHttpClient();
        response = httpClient.execute(httpGet);
        return response.getStatusLine().getStatusCode();
    } catch (Exception e) {
        e.printStackTrace();
        return 408;
    } finally {
        try {
            httpClient.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


public int getStatusCode() {
    return requestHttpsGet(url);
}
```

对 HTTP 和 HTTPS 协议的其他一些操作：

```
/**
     * 将url链接转换为ip:port形式
     *
     * @param url
     * @return
     */
    public static String convertUrlToIP(String url) {
        InetAddress ia;
        try {
            URL u = new URL(url);
            // URL u = new URL(args[0]);
            System.out.println("hostname=" + u.getHost());

            ia = InetAddress.getByName(u.getHost());
            String ip = ia.getHostAddress();
            System.out.println("IP resolved =" + ip);

            int port = u.getPort();
            if (port < 0) {
                port = u.getDefaultPort();
            }
            System.out.println("Port resolved =" + port);
            return ip + ":" + port;
        } catch (Exception e) {
            e.printStackTrace();
            return "";
        }
    }    
```

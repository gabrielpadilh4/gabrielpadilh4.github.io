---
layout: post
title:  "Java SSL Handshake - Fatal (CERTIFICATE_UNKNOWN)"
date:   2024-01-02 20:00:00 -0000
categories: java ssl tls handshake unknown certificate
author: Gabriel Padilha
---

It's common to deal with SSL/TLS exceptions when stablishing a secure connection. When using Java applications which also includes enterprise application servers, you will see exceptions caused by SSL Handshake in application logs.

The website [https://badssl.com][badssl] has many examples that can be used to test and study handshake exceptions. In my example, i will use the website [https://self-signed.badssl.com][self-signed-badssl] and the [ssl-handshake-debugger][ssl-handshake-debugger] application to explain the java exception.

# Fatal (CERTIFICATE_UNKNOWN)

Self signed certificates in general are used in internal environments where the public access is restricted. Also, a self signed certificate is not signed by a public certificate authority. By default, self signed certificates are untrusted by browser, operation system and applications. It is up to the user, administrator or developer to trust or not.

In Java, a certificate list of trusted certificate authorities comes in a bundle called `cacerts`. This is the default truststore that Java uses when stablishing SSL/TLS connections. As part of handshake, it will read the server certificate informations and try to find in `cacerts` the CA that signed that certificate. If it exists, then the handshake process continues, otherwise it will give an error like the below:

{% highlight java %}
javax.net.ssl|ERROR|01|main|2024-01-02 18:46:10.374 BRT|TransportContext.java:352|Fatal (CERTIFICATE_UNKNOWN): PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target (
"throwable" : {
  sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
  	at java.base/sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:439)
  	at java.base/sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:306)
  	at java.base/sun.security.validator.Validator.validate(Validator.java:264)
  	at java.base/sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:313)
  	at java.base/sun.security.ssl.X509TrustManagerImpl.checkTrusted(X509TrustManagerImpl.java:222)
  	at java.base/sun.security.ssl.X509TrustManagerImpl.checkServerTrusted(X509TrustManagerImpl.java:129)
  	at java.base/sun.security.ssl.CertificateMessage$T12CertificateConsumer.checkServerCerts(CertificateMessage.java:638)
  	at java.base/sun.security.ssl.CertificateMessage$T12CertificateConsumer.onCertificate(CertificateMessage.java:473)
  	at java.base/sun.security.ssl.CertificateMessage$T12CertificateConsumer.consume(CertificateMessage.java:369)
  	at java.base/sun.security.ssl.SSLHandshake.consume(SSLHandshake.java:392)
  	at java.base/sun.security.ssl.HandshakeContext.dispatch(HandshakeContext.java:443)
  	at java.base/sun.security.ssl.HandshakeContext.dispatch(HandshakeContext.java:421)
  	at java.base/sun.security.ssl.TransportContext.dispatch(TransportContext.java:183)
  	at java.base/sun.security.ssl.SSLTransport.decode(SSLTransport.java:172)
  	at java.base/sun.security.ssl.SSLSocketImpl.decode(SSLSocketImpl.java:1511)
  	at java.base/sun.security.ssl.SSLSocketImpl.readHandshakeRecord(SSLSocketImpl.java:1421)
  	at java.base/sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:456)
  	at java.base/sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:427)
  	at io.github.gabrielpadilh4.services.SSLService.openUrlSocket(SSLService.java:88)
  	at io.github.gabrielpadilh4.services.SSLService.logSSLHandshake(SSLService.java:132)
  	at io.github.gabrielpadilh4.commands.SSLDebugCommand.call(SSLDebugCommand.java:92)
  	at io.github.gabrielpadilh4.commands.SSLDebugCommand.call(SSLDebugCommand.java:14)
  	at picocli.CommandLine.executeUserObject(CommandLine.java:2041)
  	at picocli.CommandLine.access$1500(CommandLine.java:148)
  	at picocli.CommandLine$RunLast.executeUserObjectOfLastSubcommandWithSameParent(CommandLine.java:2461)
  	at picocli.CommandLine$RunLast.handle(CommandLine.java:2453)
  	at picocli.CommandLine$RunLast.handle(CommandLine.java:2415)
  	at picocli.CommandLine$AbstractParseResultHandler.execute(CommandLine.java:2273)
  	at picocli.CommandLine$RunLast.execute(CommandLine.java:2417)
  	at picocli.CommandLine.execute(CommandLine.java:2170)
  	at io.github.gabrielpadilh4.Main.main(Main.java:12)
  Caused by: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
  	at java.base/sun.security.provider.certpath.SunCertPathBuilder.build(SunCertPathBuilder.java:148)
  	at java.base/sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:129)
  	at java.base/java.security.cert.CertPathBuilder.build(CertPathBuilder.java:297)
  	at java.base/sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:434)
  	... 30 more}
{% endhighlight %}

Let's understand the anatomy of this exception. You may ask yourself, what does `unable to find valid certification path to requested target` means. In your application, set the property `-Djavax.net.debug=ssl:handshake:verbose` otherwise use the [ssl-handshake-debugger][ssl-handshake-debugger] application with the following:

{% highlight shell %}
$ ssl-handshake-debugger -u https://self-signed.badssl.com/
{% endhighlight %}

You will see the certificate informations presented by the server to the client, at the `consuming server Certificate handshake message`. In general, a server certificate presents you the certificate chain, starting by leaf, intermediary and root CA certificates. With self signed certificates, this can be a little different since there is no CA. See below:
{% highlight java %}
javax.net.ssl|DEBUG|01|main|2024-01-02 18:46:10.354 BRT|CertificateMessage.java:366|Consuming server Certificate handshake message (
"Certificates": [
  "certificate" : {
    "version"            : "v3",
    "serial number"      : "00 CA E5 12 F9 97 A8 1D 53",
    "signature algorithm": "SHA256withRSA",
    "issuer"             : "CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US",
    "not before"         : "2023-11-29 19:34:04.000 BRT",
    "not  after"         : "2025-11-28 19:34:04.000 BRT",
    "subject"            : "CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US",
    "subject public key" : "RSA",
    "extensions"         : [
      {
        ObjectId: 2.5.29.19 Criticality=false
        BasicConstraints:[
          CA:false
          PathLen: undefined
        ]
      },
      {
        ObjectId: 2.5.29.17 Criticality=false
        SubjectAlternativeName [
          DNSName: *.badssl.com
          DNSName: badssl.com
        ]
      }
    ]}
]
)
{% endhighlight %}

If you list the `cacerts`, in my case using Fedora and JDK 11, it is located in `/etc/java/java-11-openjdk/java-11-openjdk-11.0.21.0.9-3.fc39.x86_64/lib/security/cacerts`, there is no certificate with the path `CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US`. 

You can list usign the following command(it it asks for a password, the default is `changeit`):
{% highlight shell %}
$ keytool -list -v -keystore /etc/java/java-11-openjdk/java-11-openjdk-11.0.21.0.9-3.fc39.x86_64/lib/security/cacerts
{% endhighlight %}

There is two ways to fix this issue, both of them is related to importing the certificate into a truststore. The options are:
- Import the self signed certificate into cacerts
- Import the self signed certificate into a custom truststore

If you want to use the option of importing the certificate in cacerts. You need the certificate file and import with the following command:
{% highlight shell %}
$ keytool -import -alias certificate_alias -keystore /etc/java/java-11-openjdk/java-11-openjdk-11.0.21.0.9-3.fc39.x86_64/lib/security/cacerts
{% endhighlight %}

Check with the JDK vendor that you are using the right way to import the certificate, some systems or JDK distributions use a shared `cacerts` file, importing a certificate may change depending of the OS and JDK.

For importing a certificate into custom truststore, it makes use of the system properties `javax.net.ssl.trustStore` and `javax.net.ssl.trustStorePassword`, see [Oracle official documentation][oracle-truststore].

1. Get the server certificate
An system administrator can share the public certificate, otherwise you can use the following command:
{% highlight shell %}
$ openssl s_client --connect self-signed.badssl.com:443
CONNECTED(00000003)
depth=0 C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
verify error:num=18:self-signed certificate
verify return:1
depth=0 C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
   i:C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 29 22:34:04 2023 GMT; NotAfter: Nov 28 22:34:04 2025 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDeTCCAmGgAwIBAgIJAMrlEvmXqB1TMA0GCSqGSIb3DQEBCwUAMGIxCzAJBgNV
BAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRYwFAYDVQQHDA1TYW4gRnJhbmNp
c2NvMQ8wDQYDVQQKDAZCYWRTU0wxFTATBgNVBAMMDCouYmFkc3NsLmNvbTAeFw0y
MzExMjkyMjM0MDRaFw0yNTExMjgyMjM0MDRaMGIxCzAJBgNVBAYTAlVTMRMwEQYD
VQQIDApDYWxpZm9ybmlhMRYwFAYDVQQHDA1TYW4gRnJhbmNpc2NvMQ8wDQYDVQQK
DAZCYWRTU0wxFTATBgNVBAMMDCouYmFkc3NsLmNvbTCCASIwDQYJKoZIhvcNAQEB
BQADggEPADCCAQoCggEBAMIE7PiM7gTCs9hQ1XBYzJMY61yoaEmwIrX5lZ6xKyx2
PmzAS2BMTOqytMAPgLaw+XLJhgL5XEFdEyt/ccRLvOmULlA3pmccYYz2QULFRtMW
hyefdOsKnRFSJiFzbIRMeVXk0WvoBj1IFVKtsyjbqv9u/2CVSndrOfEk0TG23U3A
xPxTuW1CrbV8/q71FdIzSOciccfCFHpsKOo3St/qbLVytH5aohbcabFXRNsKEqve
ww9HdFxBIuGa+RuT5q0iBikusbpJHAwnnqP7i/dAcgCskgjZjFeEU4EFy+b+a1SY
QCeFxxC7c3DvaRhBB0VVfPlkPz0sw6l865MaTIbRyoUCAwEAAaMyMDAwCQYDVR0T
BAIwADAjBgNVHREEHDAaggwqLmJhZHNzbC5jb22CCmJhZHNzbC5jb20wDQYJKoZI
hvcNAQELBQADggEBAJYrbHPC6Yor7oi3aimJAPnnTh9Z7sQyaGfZ4I1ZIayWGIF7
+9dq/VtCYxEeq7bZELvqcK6LMtQQ7xGoJ5yCgJWjO/SbLaSy1AEa5m9im3Gg2k4w
h1AE8Z3CQUEdazVTsLKxdCp+eN62jQAzTY8xQ6yKDaWmTUhvSgErJyBv/H+vTQ+9
L5ghqMrDUZTkxgwlXs3OyJi/S/Rfv9OGiEua/T+h3yHEzOL53d+IiagOUCjUg7mP
5g4MP8zks3VcxERVjtzOahBH7fvhsMuJ/i+lSiNMMVaOr/U9Y1Y9kq96YIPax6Re
Jok9KYiYJsWbiimaCxWFT/HbLvD+qri7lD2Gm8A=
-----END CERTIFICATE-----
subject=C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
issuer=C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
---
No client certificate CA names sent
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, prime256v1, 256 bits
---
SSL handshake has read 1599 bytes and written 452 bytes
Verification error: self-signed certificate
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 5E8A7D6454E80C002D1BE296D40B910A4EDE28F713E51C5B32E55FC991639A4B
    Session-ID-ctx: 
    Master-Key: 21E3B55D6860B16530E0C163A52231439B4A4DBE6E45A341EE2D7A1E946D5F7AAB47FBCDE7D25B65BA56818799A07F1C
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 36 3d d6 15 eb 8f 4c 30-5e f7 cf 29 41 bc b6 67   6=....L0^..)A..g
    0010 - 01 47 98 63 a3 da b9 96-e5 33 20 6e 7c 12 5c 39   .G.c.....3 n|.\9
    0020 - e0 74 10 1a 1c ce 88 39-bc f6 15 8a 24 1b cc 02   .t.....9....$...
    0030 - 0e 5c 48 f6 ce ed 11 0f-12 f6 d7 77 38 1f 30 5b   .\H........w8.0[
    0040 - e5 f9 2e 10 ba c8 75 5f-87 88 c9 51 f8 0a 28 d8   ......u_...Q..(.
    0050 - 7b f2 c3 4b b1 e8 ab 4b-19 6c 16 95 d4 d3 47 00   {..K...K.l....G.
    0060 - b5 a1 b4 35 18 6f 06 85-a1 14 e7 c1 b0 e0 e5 51   ...5.o.........Q
    0070 - 09 ec 2f e1 ed 5e af 2a-f4 aa 09 2e f4 7d 50 81   ../..^.*.....}P.
    0080 - 4a 74 c7 e0 fd ad d2 37-eb 60 2f e0 c7 17 4e 9f   Jt.....7.`/...N.
    0090 - 76 26 7f 02 9b 72 ff 09-fc 7a 06 e3 2f 51 e2 ec   v&...r...z../Q..
    00a0 - bb 2f 3a 4e 02 36 6b ce-43 38 6a c3 e1 ff 61 f0   ./:N.6k.C8j...a.
    00b0 - a3 e2 dc 25 02 32 27 2f-8d 47 d9 ef 7f 6c 4a 72   ...%.2'/.G...lJr
    00c0 - cd 8f 6a 24 7d 11 9b 24-25 44 8c b0 18 3a e4 db   ..j$}..$%D...:..

    Start Time: 1704235202
    Timeout   : 7200 (sec)
    Verify return code: 18 (self-signed certificate)
    Extended master secret: no
---
^C
{% endhighlight %}

Save the contents of `-----BEGIN CERTIFICATE-----` to `-----END CERTIFICATE-----` in a file, `server.cer` for example. To check it's contents, use the command `openssl x509 -noout -text -in server.cer`, example:
{% highlight shell %}
$ openssl x509 -noout -text -in server.cer 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ca:e5:12:f9:97:a8:1d:53
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
        Validity
            Not Before: Nov 29 22:34:04 2023 GMT
            Not After : Nov 28 22:34:04 2025 GMT
        Subject: C = US, ST = California, L = San Francisco, O = BadSSL, CN = *.badssl.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c2:04:ec:f8:8c:ee:04:c2:b3:d8:50:d5:70:58:
                    cc:93:18:eb:5c:a8:68:49:b0:22:b5:f9:95:9e:b1:
                    2b:2c:76:3e:6c:c0:4b:60:4c:4c:ea:b2:b4:c0:0f:
                    80:b6:b0:f9:72:c9:86:02:f9:5c:41:5d:13:2b:7f:
                    71:c4:4b:bc:e9:94:2e:50:37:a6:67:1c:61:8c:f6:
                    41:42:c5:46:d3:16:87:27:9f:74:eb:0a:9d:11:52:
                    26:21:73:6c:84:4c:79:55:e4:d1:6b:e8:06:3d:48:
                    15:52:ad:b3:28:db:aa:ff:6e:ff:60:95:4a:77:6b:
                    39:f1:24:d1:31:b6:dd:4d:c0:c4:fc:53:b9:6d:42:
                    ad:b5:7c:fe:ae:f5:15:d2:33:48:e7:22:71:c7:c2:
                    14:7a:6c:28:ea:37:4a:df:ea:6c:b5:72:b4:7e:5a:
                    a2:16:dc:69:b1:57:44:db:0a:12:ab:de:c3:0f:47:
                    74:5c:41:22:e1:9a:f9:1b:93:e6:ad:22:06:29:2e:
                    b1:ba:49:1c:0c:27:9e:a3:fb:8b:f7:40:72:00:ac:
                    92:08:d9:8c:57:84:53:81:05:cb:e6:fe:6b:54:98:
                    40:27:85:c7:10:bb:73:70:ef:69:18:41:07:45:55:
                    7c:f9:64:3f:3d:2c:c3:a9:7c:eb:93:1a:4c:86:d1:
                    ca:85
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Subject Alternative Name: 
                DNS:*.badssl.com, DNS:badssl.com
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        96:2b:6c:73:c2:e9:8a:2b:ee:88:b7:6a:29:89:00:f9:e7:4e:
        1f:59:ee:c4:32:68:67:d9:e0:8d:59:21:ac:96:18:81:7b:fb:
        d7:6a:fd:5b:42:63:11:1e:ab:b6:d9:10:bb:ea:70:ae:8b:32:
        d4:10:ef:11:a8:27:9c:82:80:95:a3:3b:f4:9b:2d:a4:b2:d4:
        01:1a:e6:6f:62:9b:71:a0:da:4e:30:87:50:04:f1:9d:c2:41:
        41:1d:6b:35:53:b0:b2:b1:74:2a:7e:78:de:b6:8d:00:33:4d:
        8f:31:43:ac:8a:0d:a5:a6:4d:48:6f:4a:01:2b:27:20:6f:fc:
        7f:af:4d:0f:bd:2f:98:21:a8:ca:c3:51:94:e4:c6:0c:25:5e:
        cd:ce:c8:98:bf:4b:f4:5f:bf:d3:86:88:4b:9a:fd:3f:a1:df:
        21:c4:cc:e2:f9:dd:df:88:89:a8:0e:50:28:d4:83:b9:8f:e6:
        0e:0c:3f:cc:e4:b3:75:5c:c4:44:55:8e:dc:ce:6a:10:47:ed:
        fb:e1:b0:cb:89:fe:2f:a5:4a:23:4c:31:56:8e:af:f5:3d:63:
        56:3d:92:af:7a:60:83:da:c7:a4:5e:26:89:3d:29:88:98:26:
        c5:9b:8a:29:9a:0b:15:85:4f:f1:db:2e:f0:fe:aa:b8:bb:94:
        3d:86:9b:c0
{% endhighlight %}

Once that we have the public certificate, we can import it into a truststore, notice that trustores/keystores are secured by an password, you need to set and password first:
{% highlight shell %}
$  keytool -import -alias badssl -keystore truststore.jdk -file server.cer 
Enter keystore password:  
Re-enter new password: 
Owner: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Issuer: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Serial number: cae512f997a81d53
Valid from: Wed Nov 29 19:34:04 BRT 2023 until: Fri Nov 28 19:34:04 BRT 2025
Certificate fingerprints:
	 SHA1: 00:75:98:67:2A:D2:B4:BE:B6:B1:45:24:31:A2:2D:77:54:C8:28:22
	 SHA256: 3D:9B:F7:8F:76:7D:E7:49:76:09:39:9A:A6:35:6E:9A:DC:2A:47:CE:75:6F:FE:09:B9:FA:33:BE:8D:76:16:C7
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 2.5.29.19 Criticality=false
BasicConstraints:[
  CA:false
  PathLen: undefined
]

#2: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: *.badssl.com
  DNSName: badssl.com
]

Trust this certificate? [no]:  yes
Certificate was added to keystore
{% endhighlight %}

To see the truststore contents, use the following(the same password of step above):
{% highlight shell %}
$ $ keytool -list -v -keystore truststore.jdk 
Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

Alias name: badssl
Creation date: Jan 2, 2024
Entry type: trustedCertEntry

Owner: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Issuer: CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US
Serial number: cae512f997a81d53
Valid from: Wed Nov 29 19:34:04 BRT 2023 until: Fri Nov 28 19:34:04 BRT 2025
Certificate fingerprints:
	 SHA1: 00:75:98:67:2A:D2:B4:BE:B6:B1:45:24:31:A2:2D:77:54:C8:28:22
	 SHA256: 3D:9B:F7:8F:76:7D:E7:49:76:09:39:9A:A6:35:6E:9A:DC:2A:47:CE:75:6F:FE:09:B9:FA:33:BE:8D:76:16:C7
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 2048-bit RSA key
Version: 3

Extensions: 

#1: ObjectId: 2.5.29.19 Criticality=false
BasicConstraints:[
  CA:false
  PathLen: undefined
]

#2: ObjectId: 2.5.29.17 Criticality=false
SubjectAlternativeName [
  DNSName: *.badssl.com
  DNSName: badssl.com
]



*******************************************
*******************************************
{% endhighlight %}

With the truststore create, you just have to inform the the properties `javax.net.ssl.trustStore` and `javax.net.ssl.trustStorePassword`. Example:
`java -jar my-application.jar -Djavax.net.ssl.trustStore=truststore.jks -Djavax.net.ssl.trustStorePassword=abc123`

As i'm using [ssl-handshake-debugger][ssl-handshake-debugger], i will use the following for demonstration of a successfull handshake:
{% highlight java %}
$ ssl-handshake-debugger -u https://self-signed.badssl.com/ -ts /tmp/truststore.jdk -tsp abc123
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:10.565 BRT|SSLCipher.java:464|jdk.tls.keyLimits:  entry = AES/GCM/NoPadding KeyUpdate 2^37. AES/GCM/NOPADDING:KEYUPDATE = 137438953472
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:10.575 BRT|SSLCipher.java:464|jdk.tls.keyLimits:  entry =  ChaCha20-Poly1305 KeyUpdate 2^37. CHACHA20-POLY1305:KEYUPDATE = 137438953472
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.035 BRT|HandshakeContext.java:296|Ignore unsupported cipher suite: TLS_AES_256_GCM_SHA384 for TLSv1.2
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.036 BRT|HandshakeContext.java:296|Ignore unsupported cipher suite: TLS_AES_128_GCM_SHA256 for TLSv1.2
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.036 BRT|HandshakeContext.java:296|Ignore unsupported cipher suite: TLS_CHACHA20_POLY1305_SHA256 for TLSv1.2
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.069 BRT|SignatureScheme.java:295|Signature algorithm, ed25519, is not supported by the underlying providers
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.070 BRT|SignatureScheme.java:295|Signature algorithm, ed448, is not supported by the underlying providers
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.073 BRT|SignatureScheme.java:383|Ignore unsupported signature scheme: ed25519
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.073 BRT|SignatureScheme.java:383|Ignore unsupported signature scheme: ed448
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.074 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: dsa_sha256
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.074 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: dsa_sha224
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.074 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: ecdsa_sha1
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.075 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: rsa_pkcs1_sha1
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.075 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: dsa_sha1
javax.net.ssl|ALL|01|main|2024-01-02 19:52:11.075 BRT|SignatureScheme.java:402|Ignore disabled signature scheme: rsa_md5
javax.net.ssl|INFO|01|main|2024-01-02 19:52:11.075 BRT|AlpnExtension.java:178|No available application protocols
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.076 BRT|SSLExtensions.java:260|Ignore, context unavailable extension: application_layer_protocol_negotiation
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.076 BRT|SSLExtensions.java:260|Ignore, context unavailable extension: cookie
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.092 BRT|SSLExtensions.java:260|Ignore, context unavailable extension: renegotiation_info
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.092 BRT|PreSharedKeyExtension.java:633|No session to resume.
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.093 BRT|SSLExtensions.java:260|Ignore, context unavailable extension: pre_shared_key
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.095 BRT|ClientHello.java:642|Produced ClientHello handshake message (
"ClientHello": {
  "client version"      : "TLSv1.2",
  "random"              : "8A 20 00 C4 43 D3 FC FB F0 E8 5C 69 63 C6 CB 0D 75 EC B8 12 13 17 D1 0A D7 DC 05 30 73 93 1A 24",
  "session id"          : "9A 0B C9 CC 30 E4 E5 72 8C 5E 1E E9 6F D9 B6 D6 BC 96 75 78 23 97 9B 4A 53 6C 0A 58 DE E8 99 D9",
  "cipher suites"       : "[TLS_AES_256_GCM_SHA384(0x1302), TLS_AES_128_GCM_SHA256(0x1301), TLS_CHACHA20_POLY1305_SHA256(0x1303), TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384(0xC02C), TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256(0xC02B), TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256(0xCCA9), TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384(0xC030), TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256(0xCCA8), TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256(0xC02F), TLS_DHE_RSA_WITH_AES_256_GCM_SHA384(0x009F), TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256(0xCCAA), TLS_DHE_RSA_WITH_AES_128_GCM_SHA256(0x009E), TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384(0xC024), TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384(0xC028), TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256(0xC023), TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256(0xC027), TLS_DHE_RSA_WITH_AES_256_CBC_SHA256(0x006B), TLS_DHE_RSA_WITH_AES_128_CBC_SHA256(0x0067), TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA(0xC00A), TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA(0xC014), TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA(0xC009), TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA(0xC013), TLS_DHE_RSA_WITH_AES_256_CBC_SHA(0x0039), TLS_DHE_RSA_WITH_AES_128_CBC_SHA(0x0033), TLS_RSA_WITH_AES_256_GCM_SHA384(0x009D), TLS_RSA_WITH_AES_128_GCM_SHA256(0x009C), TLS_RSA_WITH_AES_256_CBC_SHA256(0x003D), TLS_RSA_WITH_AES_128_CBC_SHA256(0x003C), TLS_RSA_WITH_AES_256_CBC_SHA(0x0035), TLS_RSA_WITH_AES_128_CBC_SHA(0x002F), TLS_EMPTY_RENEGOTIATION_INFO_SCSV(0x00FF)]",
  "compression methods" : "00",
  "extensions"          : [
    "server_name (0)": {
      type=host_name (0), value=self-signed.badssl.com
    },
    "status_request (5)": {
      "certificate status type": ocsp
      "OCSP status request": {
        "responder_id": <empty>
        "request extensions": {
          <empty>
        }
      }
    },
    "supported_groups (10)": {
      "versions": [x25519, secp256r1, secp384r1, secp521r1, x448, ffdhe2048, ffdhe3072, ffdhe4096, ffdhe6144, ffdhe8192]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed]
    },
    "signature_algorithms (13)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, ecdsa_sha224, rsa_sha224]
    },
    "signature_algorithms_cert (50)": {
      "signature schemes": [ecdsa_secp256r1_sha256, ecdsa_secp384r1_sha384, ecdsa_secp521r1_sha512, rsa_pss_rsae_sha256, rsa_pss_rsae_sha384, rsa_pss_rsae_sha512, rsa_pss_pss_sha256, rsa_pss_pss_sha384, rsa_pss_pss_sha512, rsa_pkcs1_sha256, rsa_pkcs1_sha384, rsa_pkcs1_sha512, ecdsa_sha224, rsa_sha224]
    },
    "status_request_v2 (17)": {
      "cert status request": {
        "certificate status type": ocsp_multi
        "OCSP status request": {
          "responder_id": <empty>
          "request extensions": {
            <empty>
          }
        }
      }
    },
    "extended_master_secret (23)": {
      <empty>
    },
    "supported_versions (43)": {
      "versions": [TLSv1.3, TLSv1.2]
    },
    "psk_key_exchange_modes (45)": {
      "ke_modes": [psk_dhe_ke]
    },
    "key_share (51)": {
      "client_shares": [  
        {
          "named group": x25519
          "key_exchange": {
            0000: 6A 7F AA DC 5E 6C EE CB   49 D3 3D C6 6E EF 9F 05  j...^l..I.=.n...
            0010: 86 1B 71 9D 03 CF 88 61   58 AF EB B3 77 0A 62 49  ..q....aX...w.bI
          }
        },
      ]
    }
  ]
}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.323 BRT|ServerHello.java:867|Consuming ServerHello handshake message (
"ServerHello": {
  "server version"      : "TLSv1.2",
  "random"              : "8C C7 3B 1F 32 E2 AA 00 00 17 F6 C2 CD D4 EF B9 77 FC EA 6D AE 91 CA 7A DA D2 39 CE 9A 81 D4 B5",
  "session id"          : "30 A6 6B E7 16 3B D3 4C DB 9B 77 44 17 81 31 61 9C 1D 43 C2 D7 EC C0 1D 0B 9A FB A5 34 5F 34 ED",
  "cipher suite"        : "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256(0xC02F)",
  "compression methods" : "00",
  "extensions"          : [
    "server_name (0)": {
      <empty extension_data field>
    },
    "renegotiation_info (65,281)": {
      "renegotiated connection": [<no renegotiated connection>]
    },
    "ec_point_formats (11)": {
      "formats": [uncompressed, ansiX962_compressed_prime, ansiX962_compressed_char2]
    }
  ]
}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.324 BRT|SSLExtensions.java:173|Ignore unavailable extension: supported_versions
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.325 BRT|ServerHello.java:963|Negotiated protocol version: TLSv1.2
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.328 BRT|SSLExtensions.java:192|Consumed extension: renegotiation_info
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.328 BRT|SSLExtensions.java:192|Consumed extension: server_name
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.329 BRT|SSLExtensions.java:173|Ignore unavailable extension: max_fragment_length
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.329 BRT|SSLExtensions.java:173|Ignore unavailable extension: status_request
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.330 BRT|SSLExtensions.java:192|Consumed extension: ec_point_formats
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.330 BRT|SSLExtensions.java:173|Ignore unavailable extension: status_request_v2
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.331 BRT|SSLExtensions.java:163|Ignore unsupported extension: supported_versions
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.331 BRT|SSLExtensions.java:163|Ignore unsupported extension: key_share
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.332 BRT|SSLExtensions.java:192|Consumed extension: renegotiation_info
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.333 BRT|SSLExtensions.java:163|Ignore unsupported extension: pre_shared_key
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.333 BRT|SSLExtensions.java:215|Ignore impact of unsupported extension: server_name
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: max_fragment_length
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: status_request
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:215|Ignore impact of unsupported extension: ec_point_formats
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: application_layer_protocol_negotiation
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: status_request_v2
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: extended_master_secret
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: supported_versions
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: key_share
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:215|Ignore impact of unsupported extension: renegotiation_info
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.334 BRT|SSLExtensions.java:207|Ignore unavailable extension: pre_shared_key
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.336 BRT|CertificateMessage.java:366|Consuming server Certificate handshake message (
"Certificates": [
  "certificate" : {
    "version"            : "v3",
    "serial number"      : "00 CA E5 12 F9 97 A8 1D 53",
    "signature algorithm": "SHA256withRSA",
    "issuer"             : "CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US",
    "not before"         : "2023-11-29 19:34:04.000 BRT",
    "not  after"         : "2025-11-28 19:34:04.000 BRT",
    "subject"            : "CN=*.badssl.com, O=BadSSL, L=San Francisco, ST=California, C=US",
    "subject public key" : "RSA",
    "extensions"         : [
      {
        ObjectId: 2.5.29.19 Criticality=false
        BasicConstraints:[
          CA:false
          PathLen: undefined
        ]
      },
      {
        ObjectId: 2.5.29.17 Criticality=false
        SubjectAlternativeName [
          DNSName: *.badssl.com
          DNSName: badssl.com
        ]
      }
    ]}
]
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.345 BRT|ECDHServerKeyExchange.java:524|Consuming ECDH ServerKeyExchange handshake message (
"ECDH ServerKeyExchange": {
  "parameters": {
    "named group": "secp256r1"
    "ecdh public": {
      0000: 04 5C F1 D0 B5 39 C4 88   8C 37 AE FA D1 94 B0 CA  .\...9...7......
      0010: 3D B9 EA A7 02 15 A7 6A   E9 AC 8D 65 69 E5 C3 F7  =......j...ei...
      0020: E1 7D 96 06 4E 81 14 78   71 36 98 3C 85 A4 53 C1  ....N..xq6.<..S.
      0030: 60 18 5B C8 F3 C2 38 F0   57 DC 30 F0 D9 FC 8B AE  `.[...8.W.0.....
      0040: 0E                                                 .
    },
  },
  "digital signature":  {
    "signature algorithm": "rsa_pkcs1_sha512"
    "signature": {
      0000: 77 8A 61 1F FE 27 1C 28   A4 BC 11 87 22 6E 4C 64  w.a..'.(...."nLd
      0010: 87 1F 87 53 B8 AB DA AD   77 53 69 E4 A3 DC BD 76  ...S....wSi....v
      0020: 0C 5B C0 45 A7 DC 65 3A   97 15 B9 1C D3 AD CB 22  .[.E..e:......."
      0030: 13 B7 CF F5 7F 45 36 9B   F5 4A A4 9E 22 75 6F AF  .....E6..J.."uo.
      0040: 13 09 9C F3 AE 53 1D 47   0C C2 8F F3 DD A0 1E DC  .....S.G........
      0050: 83 26 8E 3E 2C 74 EB 95   12 EB FA F1 BB 18 85 5E  .&.>,t.........^
      0060: A5 29 69 6D 61 C3 8B 7A   0A 2F A8 E0 1D 01 FC 29  .)ima..z./.....)
      0070: C8 C6 7B FD 59 7D CC 51   F2 B4 74 69 26 EB A0 CC  ....Y..Q..ti&...
      0080: A9 C3 51 2A 42 2F 25 95   71 5D 9C 81 DE 6C E6 F0  ..Q*B/%.q]...l..
      0090: E3 EB 3E 35 CF B3 4C 7A   96 4B 9A 53 6C 3F BF 61  ..>5..Lz.K.Sl?.a
      00A0: C8 5E DB 52 D7 8C 27 70   18 3F 62 59 39 C3 37 B3  .^.R..'p.?bY9.7.
      00B0: 52 F4 AE 33 86 0A BD AA   1E 7E EE 50 A8 58 BC 95  R..3.......P.X..
      00C0: 0D F9 A8 2A F1 51 74 B0   08 0B AE 22 35 D5 CB 48  ...*.Qt...."5..H
      00D0: 31 73 1F AD E7 00 24 EC   C5 39 48 F7 A8 22 E6 87  1s....$..9H.."..
      00E0: D2 22 3D 47 5B 90 CB F6   11 9C 04 C8 FF A3 1B C9  ."=G[...........
      00F0: A2 17 24 0C 74 93 32 5A   EA 06 12 AC 64 1D 72 66  ..$.t.2Z....d.rf
    },
  }
}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.346 BRT|ServerHelloDone.java:151|Consuming ServerHelloDone handshake message (
<empty>
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.354 BRT|ECDHClientKeyExchange.java:410|Produced ECDHE ClientKeyExchange handshake message (
"ECDH ClientKeyExchange": {
  "ecdh public": {
    0000: 04 A0 5F 10 C3 A1 FC C8   4E A9 94 EB 30 A6 F5 5F  .._.....N...0.._
    0010: 5D 9C 7B 4B 76 70 20 77   27 73 CC 93 48 57 0C 82  ]..Kvp w's..HW..
    0020: 4D 10 43 D4 0A 18 40 D0   87 C9 7E D9 FA EC 90 9D  M.C...@.........
    0030: 0F 79 0C D8 5D 12 BB 7A   4C 14 50 4F D9 27 64 5B  .y..]..zL.PO.'d[
    0040: 24                                                 $
  },
}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.363 BRT|ChangeCipherSpec.java:115|Produced ChangeCipherSpec message
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.363 BRT|Finished.java:398|Produced client Finished handshake message (
"Finished": {
  "verify data": {
    0000: 6D B6 5A 27 22 7D 7D 30   E3 8E 00 16 
  }'}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.831 BRT|ChangeCipherSpec.java:149|Consuming ChangeCipherSpec message
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.835 BRT|Finished.java:535|Consuming server Finished handshake message (
"Finished": {
  "verify data": {
    0000: 3E 18 73 98 BD EA D2 17   08 D7 9F 76 
  }'}
)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.836 BRT|SSLSocketImpl.java:578|duplex close of SSLSocket
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.837 BRT|SSLSocketImpl.java:1741|close the underlying socket
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.837 BRT|SSLSocketImpl.java:1760|close the SSL connection (initiative)
javax.net.ssl|DEBUG|01|main|2024-01-02 19:52:11.837 BRT|SSLSocketImpl.java:838|close inbound of SSLSocket
javax.net.ssl|WARNING|01|main|2024-01-02 19:52:11.838 BRT|SSLSocketImpl.java:596|SSLSocket duplex close failed (
"throwable" : {
  java.net.SocketException: Socket is closed
  	at java.base/java.net.Socket.shutdownInput(Socket.java:1539)
{% endhighlight %}




[badssl]: https://badssl.com/
[self-signed-badssl]: https://self-signed.badssl.com/
[ssl-handshake-debugger]: https://github.com/gabrielpadilh4/ssl-handshake-debugger/
[oracle-truststore]: https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html
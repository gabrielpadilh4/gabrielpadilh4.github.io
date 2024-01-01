---
layout: post
title:  "Java SSL Handshake - Fatal (CERTIFICATE_EXPIRED)"
date:   2024-01-01 20:00:00 -0000
categories: java ssl tls handshake expired certificate
author: Gabriel Padilha
---

It's common to deal with SSL/TLS exceptions when stablishing a secure connection. When using Java applications which also includes enterprise application servers, you will see exceptions caused by SSL Handshake in application logs.

The website [https://badssl.com][badssl] has many examples that can be used to test and study handshake exceptions. In my example, i will use the website [https://expired.badssl.com][expired-badssl] and the [ssl-handshake-debugger][ssl-handshake-debugger] application to explain the java exception.

# Fatal (CERTIFICATE_EXPIRED)

If you are facing a certificate expired in your java application, you will see something similar to the following in the application logs:

{% highlight java %}
javax.net.ssl|ERROR|01|main|2024-01-01 18:04:49.620 BRT|TransportContext.java:352|Fatal (CERTIFICATE_EXPIRED): PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed (
"throwable" : {
  sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
  	at java.base/sun.security.validator.PKIXValidator.doValidate(PKIXValidator.java:369)
  	at java.base/sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:263)
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
  Caused by: java.security.cert.CertPathValidatorException: validity check failed
  	at java.base/sun.security.provider.certpath.PKIXMasterCertPathValidator.validate(PKIXMasterCertPathValidator.java:135)
  	at java.base/sun.security.provider.certpath.PKIXCertPathValidator.validate(PKIXCertPathValidator.java:224)
  	at java.base/sun.security.provider.certpath.PKIXCertPathValidator.validate(PKIXCertPathValidator.java:144)
  	at java.base/sun.security.provider.certpath.PKIXCertPathValidator.engineValidate(PKIXCertPathValidator.java:83)
  	at java.base/java.security.cert.CertPathValidator.validate(CertPathValidator.java:309)
  	at java.base/sun.security.validator.PKIXValidator.doValidate(PKIXValidator.java:364)
  	... 30 more
  Caused by: java.security.cert.CertificateExpiredException: NotAfter: Sun Apr 12 20:59:59 BRT 2015
  	at java.base/sun.security.x509.CertificateValidity.valid(CertificateValidity.java:277)
  	at java.base/sun.security.x509.X509CertImpl.checkValidity(X509CertImpl.java:669)
  	at java.base/sun.security.provider.certpath.BasicChecker.verifyValidity(BasicChecker.java:190)
  	at java.base/sun.security.provider.certpath.BasicChecker.check(BasicChecker.java:144)
  	at java.base/sun.security.provider.certpath.PKIXMasterCertPathValidator.validate(PKIXMasterCertPathValidator.java:125)
  	... 35 more}

)
{% endhighlight %}

Let's understand the anatomy of this exception. You may ask yourself, where the date `Sun Apr 12 20:59:5Sun Apr 12 20:59:59 BRT 20159 BRT 2015` is coming from. In your application, set the property `-Djavax.net.debug=ssl:handshake:verbose` otherwise use the [ssl-handshake-debugger][ssl-handshake-debugger] application with the following:

{% highlight shell %}
$ ssl-handshake-debugger -u https://expired.badssl.com/
{% endhighlight %}

You will see the certificate informations presented by the server to the client. At the `consuming server Certificate handshake message`, you will see the certificate chain, starting by leaf, intermediary and root CA certificates:

{% highlight java %}
javax.net.ssl|DEBUG|01|main|2024-01-01 18:04:49.582 BRT|CertificateMessage.java:366|Consuming server Certificate handshake message (
"Certificates": [
  "certificate" : {
    "version"            : "v3",
    "serial number"      : "4A E7 95 49 FA 9A BE 3F 10 0F 17 A4 78 E1 69 09",
    "signature algorithm": "SHA256withRSA",
    "issuer"             : "CN=COMODO RSA Domain Validation Secure Server CA, O=COMODO CA Limited, L=Salford, ST=Greater Manchester, C=GB",
    "not before"         : "2015-04-08 21:00:00.000 BRT",
    "not  after"         : "2015-04-12 20:59:59.000 BRT",
    "subject"            : "CN=*.badssl.com, OU=PositiveSSL Wildcard, OU=Domain Control Validated",
    "subject public key" : "RSA",
    "extensions"         : [
      {
        ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
        AuthorityInfoAccess [
          [
           accessMethod: caIssuers
           accessLocation: URIName: http://crt.comodoca.com/COMODORSADomainValidationSecureServerCA.crt
        , 
           accessMethod: ocsp
           accessLocation: URIName: http://ocsp.comodoca.com
        ]
        ]
      },
      {
        ObjectId: 2.5.29.35 Criticality=false
        AuthorityKeyIdentifier [
        KeyIdentifier [
        0000: 90 AF 6A 3A 94 5A 0B D8   90 EA 12 56 73 DF 43 B4  ..j:.Z.....Vs.C.
        0010: 3A 28 DA E7                                        :(..
        ]
        ]
      },
      {
        ObjectId: 2.5.29.19 Criticality=true
        BasicConstraints:[
          CA:false
          PathLen: undefined
        ]
      },
      {
        ObjectId: 2.5.29.31 Criticality=false
        CRLDistributionPoints [
          [DistributionPoint:
             [URIName: http://crl.comodoca.com/COMODORSADomainValidationSecureServerCA.crl]
        ]]
      },
      {
        ObjectId: 2.5.29.32 Criticality=false
        CertificatePolicies [
          [CertificatePolicyId: [1.3.6.1.4.1.6449.1.2.2.7]
        [PolicyQualifierInfo: [
          qualifierID: 1.3.6.1.5.5.7.2.1
          qualifier: 0000: 16 1D 68 74 74 70 73 3A   2F 2F 73 65 63 75 72 65  ..https://secure
        0010: 2E 63 6F 6D 6F 64 6F 2E   63 6F 6D 2F 43 50 53     .comodo.com/CPS
        
        ]]  ]
          [CertificatePolicyId: [2.23.140.1.2.1]
        []  ]
        ]
      },
      {
        ObjectId: 2.5.29.37 Criticality=false
        ExtendedKeyUsages [
          serverAuth
          clientAuth
        ]
      },
      {
        ObjectId: 2.5.29.15 Criticality=true
        KeyUsage [
          DigitalSignature
          Key_Encipherment
        ]
      },
      {
        ObjectId: 2.5.29.17 Criticality=false
        SubjectAlternativeName [
          DNSName: *.badssl.com
          DNSName: badssl.com
        ]
      },
      {
        ObjectId: 2.5.29.14 Criticality=false
        SubjectKeyIdentifier [
        KeyIdentifier [
        0000: 9D EE C1 7B 81 0B 3A 47   69 71 18 7D 11 37 93 BC  ......:Giq...7..
        0010: A5 1B 3F FB                                        ..?.
        ]
        ]
      }
    ]},
  "certificate" : {
    "version"            : "v3",
    "serial number"      : "2B 2E 6E EA D9 75 36 6C 14 8A 6E DB A3 7C 8C 07",
    "signature algorithm": "SHA384withRSA",
    "issuer"             : "CN=COMODO RSA Certification Authority, O=COMODO CA Limited, L=Salford, ST=Greater Manchester, C=GB",
    "not before"         : "2014-02-11 22:00:00.000 BRST",
    "not  after"         : "2029-02-11 20:59:59.000 BRT",
    "subject"            : "CN=COMODO RSA Domain Validation Secure Server CA, O=COMODO CA Limited, L=Salford, ST=Greater Manchester, C=GB",
    "subject public key" : "RSA",
    "extensions"         : [
      {
        ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
        AuthorityInfoAccess [
          [
           accessMethod: caIssuers
           accessLocation: URIName: http://crt.comodoca.com/COMODORSAAddTrustCA.crt
        , 
           accessMethod: ocsp
           accessLocation: URIName: http://ocsp.comodoca.com
        ]
        ]
      },
      {
        ObjectId: 2.5.29.35 Criticality=false
        AuthorityKeyIdentifier [
        KeyIdentifier [
        0000: BB AF 7E 02 3D FA A6 F1   3C 84 8E AD EE 38 98 EC  ....=...<....8..
        0010: D9 32 32 D4                                        .22.
        ]
        ]
      },
      {
        ObjectId: 2.5.29.19 Criticality=true
        BasicConstraints:[
          CA:true
          PathLen:0
        ]
      },
      {
        ObjectId: 2.5.29.31 Criticality=false
        CRLDistributionPoints [
          [DistributionPoint:
             [URIName: http://crl.comodoca.com/COMODORSACertificationAuthority.crl]
        ]]
      },
      {
        ObjectId: 2.5.29.32 Criticality=false
        CertificatePolicies [
          [CertificatePolicyId: [2.5.29.32.0]
        []  ]
          [CertificatePolicyId: [2.23.140.1.2.1]
        []  ]
        ]
      },
      {
        ObjectId: 2.5.29.37 Criticality=false
        ExtendedKeyUsages [
          serverAuth
          clientAuth
        ]
      },
      {
        ObjectId: 2.5.29.15 Criticality=true
        KeyUsage [
          DigitalSignature
          Key_CertSign
          Crl_Sign
        ]
      },
      {
        ObjectId: 2.5.29.14 Criticality=false
        SubjectKeyIdentifier [
        KeyIdentifier [
        0000: 90 AF 6A 3A 94 5A 0B D8   90 EA 12 56 73 DF 43 B4  ..j:.Z.....Vs.C.
        0010: 3A 28 DA E7                                        :(..
        ]
        ]
      }
    ]},
  "certificate" : {
    "version"            : "v3",
    "serial number"      : "27 66 EE 56 EB 49 F3 8E AB D7 70 A2 FC 84 DE 22",
    "signature algorithm": "SHA384withRSA",
    "issuer"             : "CN=AddTrust External CA Root, OU=AddTrust External TTP Network, O=AddTrust AB, C=SE",
    "not before"         : "2000-05-30 07:48:38.000 BRT",
    "not  after"         : "2020-05-30 07:48:38.000 BRT",
    "subject"            : "CN=COMODO RSA Certification Authority, O=COMODO CA Limited, L=Salford, ST=Greater Manchester, C=GB",
    "subject public key" : "RSA",
    "extensions"         : [
      {
        ObjectId: 1.3.6.1.5.5.7.1.1 Criticality=false
        AuthorityInfoAccess [
          [
           accessMethod: ocsp
           accessLocation: URIName: http://ocsp.usertrust.com
        ]
        ]
      },
      {
        ObjectId: 2.5.29.35 Criticality=false
        AuthorityKeyIdentifier [
        KeyIdentifier [
        0000: AD BD 98 7A 34 B4 26 F7   FA C4 26 54 EF 03 BD E0  ...z4.&...&T....
        0010: 24 CB 54 1A                                        $.T.
        ]
        ]
      },
      {
        ObjectId: 2.5.29.19 Criticality=true
        BasicConstraints:[
          CA:true
          PathLen:2147483647
        ]
      },
      {
        ObjectId: 2.5.29.31 Criticality=false
        CRLDistributionPoints [
          [DistributionPoint:
             [URIName: http://crl.usertrust.com/AddTrustExternalCARoot.crl]
        ]]
      },
      {
        ObjectId: 2.5.29.32 Criticality=false
        CertificatePolicies [
          [CertificatePolicyId: [2.5.29.32.0]
        []  ]
        ]
      },
      {
        ObjectId: 2.5.29.15 Criticality=true
        KeyUsage [
          DigitalSignature
          Key_CertSign
          Crl_Sign
        ]
      },
      {
        ObjectId: 2.5.29.14 Criticality=false
        SubjectKeyIdentifier [
        KeyIdentifier [
        0000: BB AF 7E 02 3D FA A6 F1   3C 84 8E AD EE 38 98 EC  ....=...<....8..
        0010: D9 32 32 D4                                        .22.
        ]
        ]
      }
    ]}
]
)
{% endhighlight %}

As we can see, at the leaf certificate, it is valid "not before" 2015-04-08 and "not after" 2015-04-12:

{% highlight java %}
    "issuer"             : "CN=COMODO RSA Domain Validation Secure Server CA, O=COMODO CA Limited, L=Salford, ST=Greater Manchester, C=GB",
    "not before"         : "2015-04-08 21:00:00.000 BRT",
    "not  after"         : "2015-04-12 20:59:59.000 BRT",
    "subject"            : "CN=*.badssl.com, OU=PositiveSSL Wildcard, OU=Domain Control Validated",
    "subject public key" : "RSA",
{% endhighlight %}

In this type of scenario you should reach the server administrator to renew it's certificate to a valid date. If the certificate is not valid, the communication will always fail with this exception.

If you are like me, someone who wants to see with your own eyes where this validation is done in the code, see below the method `valid` of [CertificateValidity][CertificateValidity] class for OpenJDK 11(most of JDK distributions uses the same code):

{% highlight java %}
    /**
     * Verify that the passed time is within the validity period.
     * @param now the Date against which to compare the validity
     * period.
     *
     * @exception CertificateExpiredException if the certificate has expired
     * with respect to the <code>Date</code> supplied.
     * @exception CertificateNotYetValidException if the certificate is not
     * yet valid with respect to the <code>Date</code> supplied.
     *
     */
    public void valid(Date now)
    throws CertificateNotYetValidException, CertificateExpiredException {
        /*
         * we use the internal Dates rather than the passed in Date
         * because someone could override the Date methods after()
         * and before() to do something entirely different.
         */
        if (notBefore.after(now)) {
            throw new CertificateNotYetValidException("NotBefore: " +
                                                      notBefore.toString());
        }
        if (notAfter.before(now)) {
            throw new CertificateExpiredException("NotAfter: " +
                                                  notAfter.toString());
        }
    }
{% endhighlight %}


[badssl]: https://badssl.com/
[expired-badssl]: https://expired.badssl.com
[ssl-handshake-debugger]: https://github.com/gabrielpadilh4/ssl-handshake-debugger/
[CertificateValidity]: https://github.com/openjdk/jdk11/blob/37115c8ea4aff13a8148ee2b8832b20888a5d880/src/java.base/share/classes/sun/security/x509/CertificateValidity.java#L262C19-L262C19

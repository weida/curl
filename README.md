# Curl with TASSL support for TLCP.


## build

Specify the prefix as /root/install , and you can modify it according to your enviroment, Compile using static linking, tassl_tlcp.patch is located in the /root/install.

`NOTE: Currently, testing is only conducted on Linux`


### TASSL
  base on openssl1.1.1s https://github.com/jntass/TASSL-1.1.1/

    ./config --prefix=/root/install/tassl111s -DOPENSSL_TLS_SECURITY_LEVEL=0 enable-rc4 enable-des enable-ssl2 enable-ssl3 enable-ssl3-method enable-weak-ssl-ciphers no-shared
    make
    make install

### nghttp2
  nghttp2-1.59.0 https://github.com/nghttp2/nghttp2/

    autoreconf -i
    automake
    autoconf
    ./configure --prefix=/root/install/nghttp2 --disable-shared --enable-static
    make
    make install

### libpsl
  libpsl-0.21.5 https://github.com/rockdaboot/libpsl

    ./configure --prefix=/root/install/libpsl --disable-shared --enable-static
    make 
    make install

### curl 
  curl-8.6.0 https://github.com/curl/curl

    patch -p1 < ../tassl_tlcp.patch
    CPPFLAGS="-I/root/install/libpsl/include" LDFLAGS="-L/root/install/libpsl/lib" ./configure --prefix=/root/install/curl --disable-shared --enable-static --with-ssl=/root/install/tassl111s --with-nghttp2=/root/install/nghttp2
    make 
    make install


## usage

  add extra option:

    --cert2 <certificate[:password]> SM encryption Client certificate file and password
    --cert2-type <type> SM encryption Certificate type (DER/PEM/ENG/P12)
    --key2 <key>  SM encryption Private key file name
    --key2-type <type> SM encryption Private key file type (DER/PEM/ENG)
    --pass2 <phrase> Pass phrase for the SM encryption private key
    --tlcp        Use TLCP

```
./curl --tlcp -k https://demo.gmssl.cn:1443 -v
* Host demo.gmssl.cn:1443 was resolved.
* IPv6: (none)
* IPv4: 47.93.114.141
*   Trying 47.93.114.141:1443...
* Connected to demo.gmssl.cn (47.93.114.141) port 1443
* ALPN: curl offers h2,http/1.1
* TLCP (OUT), TLS handshake, Client hello (1):
* TLCP (IN), TLS handshake, Server hello (2):
* TLCP (IN), TLS handshake, Certificate (11):
* TLCP (IN), TLS handshake, Server key exchange (12):
* TLCP (IN), TLS handshake, Server finished (14):
* TLCP (OUT), TLS handshake, Client key exchange (16):
* TLCP (OUT), TLS change cipher, Change cipher spec (1):
* TLCP (OUT), TLS handshake, Finished (20):
* TLCP (IN), TLS handshake, Finished (20):
* SSL connection using GMTLSv1.1 / ECC-SM4-GCM-SM3 / [blank] / UNDEF
* ALPN: server did not agree on a protocol. Uses default.
* Server certificate:
*  subject: C=CN; CN=demo.gmssl.cn
*  start date: Dec  3 16:00:00 2023 GMT
*  expire date: Dec  3 16:00:00 2028 GMT
*  issuer: C=CN; O=GMSSL; OU=PKI/SM2; CN=MiddleCA for Test
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
*   Certificate level 0: Public key type ? (256/128 Bits/secBits), signed using sm3WithSM2Sign
*   Certificate level 1: Public key type ? (256/128 Bits/secBits), signed using sm3WithSM2Sign
* using HTTP/1.x
> GET / HTTP/1.1
> Host: demo.gmssl.cn:1443
> User-Agent: curl/8.6.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0
< Date: Fri, 16 Feb 2024 03:29:33 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Wed, 07 Feb 2024 12:17:59 GMT
< Connection: keep-alive
< ETag: "65c374f7-267"
< Accept-Ranges: bytes
...
```

test certificates from  https://www.gmcrt.cn/gmcrt/index.jsp

```
./curl --tlcp --cert sm2.user01.sig.crt.pem --key sm2.user01.sig.key.pem --cert2 sm2.user01.enc.crt.pem --key2 sm2.user01.enc.key.pem --cacert sm2.ca.pem https://demo.gmssl.cn:2443/ -v
* Host demo.gmssl.cn:2443 was resolved.
* IPv6: (none)
* IPv4: 47.93.114.141
*   Trying 47.93.114.141:2443...
* Connected to demo.gmssl.cn (47.93.114.141) port 2443
* ALPN: curl offers h2,http/1.1
* TLCP (OUT), TLS handshake, Client hello (1):
*  CAfile: sm2.ca.pem
*  CApath: none
* TLCP (IN), TLS handshake, Server hello (2):
* TLCP (IN), TLS handshake, Certificate (11):
* TLCP (IN), TLS handshake, Server key exchange (12):
* TLCP (IN), TLS handshake, Request CERT (13):
* TLCP (IN), TLS handshake, Server finished (14):
* TLCP (OUT), TLS handshake, Certificate (11):
* TLCP (OUT), TLS handshake, Client key exchange (16):
* TLCP (OUT), TLS handshake, CERT verify (15):
* TLCP (OUT), TLS change cipher, Change cipher spec (1):
* TLCP (OUT), TLS handshake, Finished (20):
* TLCP (IN), TLS handshake, Finished (20):
* SSL connection using GMTLSv1.1 / ECC-SM4-GCM-SM3 / [blank] / UNDEF
* ALPN: server did not agree on a protocol. Uses default.
* Server certificate:
*  subject: C=CN; CN=demo.gmssl.cn
*  start date: Dec  3 16:00:00 2023 GMT
*  expire date: Dec  3 16:00:00 2028 GMT
*  subjectAltName: host "demo.gmssl.cn" matched cert's "demo.gmssl.cn"
*  issuer: C=CN; O=GMSSL; OU=PKI/SM2; CN=MiddleCA for Test
*  SSL certificate verify ok.
*   Certificate level 0: Public key type ? (256/128 Bits/secBits), signed using sm3WithSM2Sign
*   Certificate level 1: Public key type ? (256/128 Bits/secBits), signed using sm3WithSM2Sign
* using HTTP/1.x
> GET / HTTP/1.1
> Host: demo.gmssl.cn:2443
> User-Agent: curl/8.6.0
> Accept: */*
...

```





<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [HTTPS webservice](#https-webservice)
- [Set up HTTPS web service](#set-up-https-web-service)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# HTTPS webservice

[JAN072021-IBM IIB REST_GET_SSL_DB 20:50](https://drive.google.com/file/d/1o6TpH129InsCw8LWumlPOqUrfjgZU7O_/view?usp=share_link). IIB you can either consume
HTTPS webservice or expose it. HTTPS is a transport layer security. When
you are exposing the service, you are the provider. You will have a
keystore. When you are consuming you are the client, and you will have a
truststore.

In ikeyman create a keystore of type JKS called tstore.jks.

(JKS, Java Key Store. You can find this file at
sun.security.provider.JavaKeyStore. This keystore is Java specific, it
usually has an extension of jks. This type of keystore can contain
private keys and certificates, but it cannot be used to store secret
keys. Since it's a Java specific keystore, so it cannot be used in
other programming languages. The private keys stored in JKS cannot be
extracted in Java.)
The tstore.jks is protected by a password. If we are planning to use
tstore.jks as a keystore (since we are the provider) then we choose
personal certificate from the ikeyman dropdown. If we are using the file
as a truststore (since we are consuming a service) we choose Signer
certificates.

![Certificate Authority](images/ca-diagram-b.png)

# Set up HTTPS web service

[ ]

You have to set up SSL in the JVM settings of the integration server
via the ComIbmJVMManager object.

The ikeyman utility allows you to create a keystore database file of
type JKS. Call it iibkeystore.jks and provide a password.

[The keystore initially only contains a private key.]

[ ]

Click on new self-signed certificate with key label personalcert then
select it then click on validate. The certificate is contained within
the keystore. You then need to associate the keystore with the execution
group.

[ ]

[https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiWyMPF9qPvAhW1WxUIHSSSD_cQFjAAegQIBBAD&url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FSelf-signed_certificate&usg=AOvVaw1ulmEXUANNETSV5MnkQghm[]{.s15}](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiWyMPF9qPvAhW1WxUIHSSSD_cQFjAAegQIBBAD&url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FSelf-signed_certificate&usg=AOvVaw1ulmEXUANNETSV5MnkQghm)

[ ]

+---------------------------------------------------------------------+
| [Commands to associate keystore with your execution group.] |
+---------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|  |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| mqsichangeproperties IIBGURU -e IIBGURU_EX -o ComIbmCacheManager -n |
| explicitlySetPort -v 7788 |
| |
| mqsichangeproperties IIBGURU -e IIBGURU_EX -o ComIbmJVMManager -n |
| keystoreFile -v |
| /home/mqbrkruser/iib-10.0.0.22/MiddlewareGuy/iibkeystore.jks |
| |
| mqsichangeproperties IIBGURU -e IIBGURU_EX -o ComIbmJVMManager -n |
| keystoreType -v JKS |
| |
| mqsichangeproperties IIBGURU -e IIBGURU_EX -o ComIbmJVMManager -n |
| keystorePass -v defaultKeystore::password |
| |
| [ ] |
| |
| mqsisetdbparms IIBGURU -n defaultKeystore::password -u ignore -p |
| G8n35hj1 |
| |
| [mqsireload IIBGURU -e IIBGURU_EX] |
+-----------------------------------------------------------------------+

[ ]

[ ]

[ ]

[ ]


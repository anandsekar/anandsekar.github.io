---
layout: post
title: "Exporting the Private Key From a JKS Keystore"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2006-01-19T05:21:31-07:00
---
A common problem faced when moving certificates and keys from tomcat to Apache web server is that keytool does not allow you to export the private key in the format that apache’s modssl module requires. [Mark Foster’s](http://mark.foster.cc/kb/openssl-keytool.html) post and [Andrew Morrow’s](http://forum.java.sun.com/thread.jspa?forumID=2&messageID=449486&threadID=154587) post contains valuable information on how to export a key from a JKS keystore.

Here is a summary of the steps needed to export a private key
Download [ExportPrivateKey.zip]({{site.url}}/assets/downloads/pkexport/ExportPrivateKey.zip)
Invoke
{% highlight bash %}
	java -jar ExportPrivateKey.zip {keystore_path} JKS {keystore_password} {alias} {target_file}
{% endhighlight %}
This would export the key to PKCS #8 PEM format. Now run openssl to convert it to the format apache modssl expects the file in
{% highlight bash %}
	openssl pkcs8 -inform PEM -nocrypt -in exported-pkcs8.key -out exported.key
{% endhighlight %}
The java code for exporting the private key in PKCS #8 format
{% highlight java %}
	import java.io.File;
import java.io.FileInputStream;
import java.io.FileWriter;
import java.security.Key;
import java.security.KeyPair;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.UnrecoverableKeyException;
import java.security.cert.Certificate;
 
import sun.misc.BASE64Encoder;
 
public class ExportPrivateKey {
        private File keystoreFile;
        private String keyStoreType;
        private char[] password;
        private String alias;
        private File exportedFile;
 
        public static KeyPair getPrivateKey(KeyStore keystore, String alias, char[] password) {
                try {
                        Key key=keystore.getKey(alias,password);
                        if(key instanceof PrivateKey) {
                                Certificate cert=keystore.getCertificate(alias);
                                PublicKey publicKey=cert.getPublicKey();
                                return new KeyPair(publicKey,(PrivateKey)key);
                        }
                } catch (UnrecoverableKeyException e) {
        } catch (NoSuchAlgorithmException e) {
        } catch (KeyStoreException e) {
        }
        return null;
        }
 
        public void export() throws Exception{
                KeyStore keystore=KeyStore.getInstance(keyStoreType);
                BASE64Encoder encoder=new BASE64Encoder();
                keystore.load(new FileInputStream(keystoreFile),password);
                KeyPair keyPair=getPrivateKey(keystore,alias,password);
                PrivateKey privateKey=keyPair.getPrivate();
                String encoded=encoder.encode(privateKey.getEncoded());
                FileWriter fw=new FileWriter(exportedFile);
                fw.write(“—–BEGIN PRIVATE KEY—–\n“);
                fw.write(encoded);
                fw.write(“\n“);
                fw.write(“—–END PRIVATE KEY—–”);
                fw.close();
        }
 
 
        public static void main(String args[]) throws Exception{
                ExportPrivateKey export=new ExportPrivateKey();
                export.keystoreFile=new File(args[0]);
                export.keyStoreType=args[1];
                export.password=args[2].toCharArray();
                export.alias=args[3];
                export.exportedFile=new File(args[4]);
                export.export();
        }
}
{% endhighlight %}
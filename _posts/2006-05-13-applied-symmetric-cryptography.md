---
layout: post
title: "Applied Symmetric Cryptography"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2006-05-13T05:17:47-07:00
---

Cryptography in essence tries to solve three main real world problems.

1. Confidentialty
2. Integrity
3. Authenticity

Lets consider there are two people Alice and Bob who want to communicate with each other, and Eve is trying to eaves drop on the communication. The communication is said to be confidential if eve is not able to understand what messages Alice and Bob are exchanging. Ok, Eve cannot understand what Alice and Bob are talking, however Eve could stand between Alice and Bob and tamper the message. Somehow Bob must be able to deduce that the messages that come from Alice have not been tampered. There is yet another problem, Eve could be sending messages on behalf of Alice, somehow Bob must be able to authenticate that the message came from Alice.

There are two kinds of Cryptography

1. Symmetric/Secret Key Cryptography
2. Asymmetric/Public Key Cryptography

### Confidentiality using Symmetric/Secret Key Cryptography

Secret key cryptography, uses the same key to encrypt and decrypt the message. If Alice wants to communicate a confidential message to Bob they must first have the same encryption key in their possession.


![Confidentiality]({{ site.url }}/assets/images/appliedcrypt/confidential.png)


In the example below we use the key generator to generate a key for the DES encryption algorithm. The cipher class is then instantiated for encryption and decryption using the key which is distributed to Alice and Bob.

{% highlight java linenos%}
package org.symmetric;
 
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
 
public class ConfidentialitySecretKey {
 
        public ConfidentialitySecretKey() throws Exception {
                KeyGenerator keyGenerator = KeyGenerator.getInstance(“DES”);
                SecretKey key = keyGenerator.generateKey();
                Alice alice = new Alice();
                Bob bob = new Bob();
                alice.setKey(key);
                bob.setKey(key);
                byte[] cipherText = alice.getMessage();
                bob.receiveMessage(cipherText);
        }
 
        public class Alice {
                private SecretKey key;
 
                public void setKey(SecretKey key) {
                        this.key = key;
                }
 
                public byte[] getMessage() throws Exception {
                        String message = “Top Secret”;
                        Cipher cipher = Cipher.getInstance(“DES/ECB/PKCS5Padding”);
                        cipher.init(Cipher.ENCRYPT_MODE, key);
                        return cipher.doFinal(message.getBytes(“UTF-8″));
                }
 
        }
 
        public class Bob {
                private SecretKey key;
 
                public void setKey(SecretKey key) {
                        this.key = key;
                }
 
                public void receiveMessage(byte[] message) throws Exception {
                        Cipher cipher = Cipher.getInstance(“DES/ECB/PKCS5Padding”);
                        cipher.init(Cipher.DECRYPT_MODE, key);
                        byte[] plainText = cipher.doFinal(message);
                        System.out.println(new String(plainText, “UTF-8″));
                }
        }
 
        public static void main(String args[]) throws Exception {
                ConfidentialitySecretKey c = new ConfidentialitySecretKey();
        }
}
{% endhighlight %}

###Integrity using Symmetric/Secret Key Cryptography
Now since Alice can send a confidential message to Bob, she is faced with another problem. She needs to some how ensure the integrity of the message. On the event that the message is tampered by eve, somehow Bob should be able to detect that the message has been tampered.
A message digest is an function on the message that generates a unique but shorter representation of the message itself. Alice now generates a digest of the message that she needs to send to Bob. Now she encrypts the message digest with the secret key that only she and Bob shares. The result of the transforation is called the Message Authentication Code or MAC. She now sends the message and the MAC to Bob. Since Bob is the only other person in the world who is in possession of the secret key, nobody can tamper the message and regenerate the MAC. When Bob received the message he verifies that the message has not been tampered by generating the message digest from the message and matching it with the decripted MAC.


![Integrity]({{ site.url }}/assets/images/appliedcrypt/integrity.png)


The example below shows how one could generate a MAC using java.

{% highlight java linenos%}
package org.symmetric;
 
import java.util.Arrays;
 
import javax.crypto.KeyGenerator;
import javax.crypto.Mac;
import javax.crypto.SecretKey;
 
public class IntegritySecretKey {
 
        public IntegritySecretKey() throws Exception {
                KeyGenerator keyGen = KeyGenerator.getInstance(“HmacMD5″);
                SecretKey key = keyGen.generateKey();
                Alice alice = new Alice();
                Bob bob = new Bob();
                alice.setKey(key);
                bob.setKey(key);
                byte[] message = alice.getMessage();
                byte[] mac = alice.getMAC();
                bob.receiveMessage(message, mac);
                // eve tampers the next message
                message[0] = 0;
                bob.receiveMessage(message, mac);
        }
 
        public class Alice {
                private SecretKey key;
 
                public void setKey(SecretKey key) {
                        this.key = key;
                }
 
                public byte[] getMessage() throws Exception {
                        return “Top Secret”.getBytes(“UTF-8″);
                }
 
                public byte[] getMAC() throws Exception {
                        byte[] message = getMessage();
                        Mac mac = Mac.getInstance(key.getAlgorithm());
                        mac.init(key);
                        return mac.doFinal(message);
                }
 
        }
 
        public class Bob {
                private SecretKey key;
 
                public void setKey(SecretKey key) {
                        this.key = key;
                }
 
                public void receiveMessage(byte[] message, byte[] macReceived)
                                throws Exception {
                        Mac mac = Mac.getInstance(key.getAlgorithm());
                        mac.init(key);
                        byte[] macToCompare = mac.doFinal(message);
                        System.out.println(Arrays.equals(macToCompare, macReceived));
                }
        }
 
        public static void main(String args[]) throws Exception {
                IntegritySecretKey i = new IntegritySecretKey();
        }
}
{% endhighlight %}

### Authentication using Secret/Symmetric Key Cryptography
Using message authentication codes inherently solves the problems of authentication, as not only the integrity of the message is ensured, since Alice and Bob are the only two persons who are in possession of the secret key, if the MAC is verified it also authencitates that Alice really sent the message. Now Eve has a few tricks up her sleeve ! She could record the messages and replay them at a later time, or she should reorder the messages. This could be solved by embedding the timestamp in the messages. Eve could still cause trouble by deleting the messages totatally ! This problem cannot be solved by cryptography but with the handshake and protocol used for communication.
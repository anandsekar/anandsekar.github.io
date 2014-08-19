---
layout: post
title: "Java Serialization Versioning With Externalizable"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2009-01-05T16:59:12-07:00
---

Java Serialization is a easy way to persist the state of Java Objects. However if the structure of the class changes the serialized bytes is invalidated by the changes. This article suggest how one could implement serialization with Externalizable with support for versions.

Consider a simple java class
{% highlight java %}
public class SimpleObject implements Serializable {
        private int field1;
        private int field2;
}
{% endhighlight %}
If the structure of the class changes, then the serialized bytes becomes incompatible and you would get the dreaded ClassCastException
{% highlight java %}
public class SimpleObject implements Serializable {
        //private int field1; removed
        private int field2;
        private int field3; //added
}
{% endhighlight %}
A good way to get complete control of the serialization and support version is by implementing Externalizable instead of Serializable.

Externalizable gives the programmer full control over how the class is serialized and de-serialized. It is common practice to add a version variable to support representations of the same class over time.

Consider first version of the same class
{% highlight java %}
public class SimpleObject implements Externalizable {
        private int version=1;
        private int field1;
        private int field2;
        @Override
        public void readExternal(ObjectInput in) throws IOException,
                        ClassNotFoundException {
                version=in.readInt();
                if(version==1) {
                        field1=in.readInt();
                        field2=in.readInt();
                } else {
                        throw new IOException(“Invalid Version”);
                }
        }
        @Override
        public void writeExternal(ObjectOutput out) throws IOException {
                out.writeInt(version);
                out.writeInt(field1);
                out.writeInt(field2);
        }
}
{% endhighlight %}
Now if the next version of the class needs to add a String and remove field2 the class would look like
{% highlight java %}
public class SimpleObject implements Externalizable {
        private int version=1;
        private int field1;
        //private int field2; removed
        private String version2String;
        @Override
        public void readExternal(ObjectInput in) throws IOException,
                        ClassNotFoundException {
                version=in.readInt();
                if(version==1) {
                        field1=in.readInt();
                        in.readInt();// ignore
                        version2String=“”;// initialize to default
                } else if (version==2) {
                        field1=in.readInt();
                        version2String=(String) in.readObject();
                } else {
                        throw new IOException(“Invalid Version”);
                }
        }
        @Override
        public void writeExternal(ObjectOutput out) throws IOException {
                out.writeInt(version);
                out.writeInt(field1);
                out.writeObject(version2String);
        }
}
{% endhighlight %}

---
title: "Generating a SAN Certificate in Ruby"
layout: post
date: 2015-03-29 17:53:31
comments: true
share: true
tags: [openssl,ruby,san,certificate]
---

## Table of Contents

* [Introduction](#introduction)
* [Assumptions and Prerequisites](#assumptions-and-prerequisites)
* [Creating our Certificate Request](#creating-our-certificate-request)
  * [Including our Requirements](#including-our-requirements)
  * [Generating the Key Pair](#generating-the-key-pair)
  * [Generate the Request](#generate-the-request)
    * [Add our Extensions](#add-our-extensions)
  * [Sign our Request](#sign-our-request)
* [Code for this Example](#code-for-this-example)
* [Real World Example](#real-world-example)

# Introduction

After [Heartbleed](http://heartbleed.com/), I found myself in need of replacing
a large number of SSL keypairs, most of which included SAN certificates. Of
course, the first thing I did was try to script the process, which resulted in
some bashing of my head against my desk as I stumbled through the OpenSSL Ruby
library.

But fret not, I'll try to explain it as best I can, and if you think I've made a
mistake, I'm sure you will let me know in the comments below!

# Assumptions and Prerequisites

I assume you are using a modern Ruby, 2.1 or greater in this case. Though older
versions may work, I have not tested it out. Let me know in the comments if you
find it works/doesn't for another version.

As far as external gems, the only one we will need is the `openssl-extensions`
gem.

# Creating our Certificate Request

## Including our Requirements

I may be in the minority, but I hate when people do not give me at least a basic
idea of the require statements they needed. Since this is my article, I will do
future me a favor:

{% highlight ruby %}
require 'openssl'
require 'openssl-extensions/all'
{% endhighlight %}


## Generating the Key Pair

Now, we will generate our key pair. As you probably know, we will provide the
public key as part of our request, signing it with the private key before
having it signed by a real CA.

{% highlight ruby %}
keyfile = '/tmp/mycert.key'

key = OpenSSL::PKey::RSA.new 2048

file = File.new(keyfile,'w',0400)
file.write(key)
file.close
{% endhighlight %}

## Generate the Request

Easy, right? Next up we will generate our request object. To do that, we first
need to create our certificate subject as an `OpenSSL::X509::Name` object:

{% highlight ruby %}
subj_arr = [ ['CN', 'myhost.example.com'], [ 'DC','example'], ['DC','com']]
subj      = OpenSSL::X509::Name.new(subj_arr)
{% endhighlight %}

Next up, we will create our request:

{% highlight ruby %}
request = OpenSSL::X509::Request.new
request.version = 0
request.subject = subj
request.public_key = key.public_key
{% endhighlight %}


### Add our Extensions

Now that we have our request, we need to setup our extensions and add them to
our request. This is the critical piece of this post, since our SAN values are
one of the extensions we need to add.

To begin, I found the following to be needed for basic SSL certificates. You may
find different for your needs.

{% highlight ruby %}
exts = [
  [ 'basicConstraints', 'CA:FALSE', false ],
  [ 'keyUsage', 'Digital Signature, Non Repudiation, Key Encipherment', false ],
]
{% endhighlight %}

Next we add our SAN extension to the request. First we need to format each
SAN entry, then we'll add them to our extension array:

{% highlight ruby %}
sans = [ 'example.com', 'www.example.com' ]
sans.map! do |san|
  san = "DNS:#{san}"
end
exts << [ 'subjectAltName', sans.join(','), false ]
{% endhighlight %}

Now we need to convert our array into OpenSSL attributes, and add them to
our request.

{% highlight ruby %}
ef = OpenSSL::X509::ExtensionFactory
exts.map! do |ext|
  ef.create_extension(*ext)
end
attrval = OpenSSL::ASN1::Set([OpenSSL::ASN1::Sequence(exts)])
attrs = [
  OpenSSL::X509::Attribute.new('extReq',attrval),
  OpenSSL::X509::Attribute.new('msExtReq',attrval),
]
attrs.each do |attr|
  request.add_attribute(attr)
end
{% endhighlight %}

## Sign our Request

Finally, the very last thing we do is sign our request after we are done
modifying it. If you do any other work on the request object in your own code,
you need to make sure you do it before you get here.

{% highlight ruby %}
request.sign(key, OpenSSL::Digest::SHA1.new)

# save our request to a file
csrfile = '/tmp/csrfile'
file = File.new(csrfile,'w',0400)
file.write(request)
file.close

# print out our request to screen for good measure
puts request.to_text
{% endhighlight %}

# Code for this Example

{% gist arusso/d5a3195773c2ca3717d4 %}

# Real World Example

Remember how I needed to write a tool in the face of the Heartbleed scramble?
Well you can check out how I used the above code to write a tool that grabs an
existing certificate and extract the information I need to generate a new
key/certificate request based on it.

link: [regenerate-cert](https://github.com/arusso/cert-tools/blob/master/regenerate-cert.rb)

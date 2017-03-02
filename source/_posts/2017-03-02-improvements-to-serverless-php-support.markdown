---
layout: post
title: "Improvements to Serverless PHP support"
date: 2017-03-02 11:37
comments: true
categories: [aws, lambda, serverless, php]
---
I was inspired by two events to jump back into serverless framework. 

{% img left /images/blog/serverless-php-improvements-001.jpeg 200 Serverless London Meetup %} 

Firstly, I attended the second [London serverless meetup](https://www.meetup.com/Serverless-London/) yesterday evening which was excellent and showed just how much enthusiasm there is for serverless architectures. Check out their new logo on the left. It was significant that each of the three speakers announced that they are actively hiring serverless developers.

Secondly, [Stolz](https://github.com/Stolz) has contributed improvements to my sample project for integrating PHP into the serverless framework. It's the purpose of this blog post to cover the changes.

The trick to getting AWS lambda to support PHP is to bundle in a PHP binary so that nodejs can call it with `child_process.spawn()`. In [my first implementation](/the-serverless-framework-and-php/), I used an Ubuntu docker base image to compile and produce the php binary. Unfortunately, this is not identical to the container that AWS Lambda uses and so sometimes the logs would contain errors such as:

```sh
START RequestId: 728dcddf-feaa-11e6-8346-2125e1c055d7 Version: $LATEST
2017-03-01 18:11:10.455 (+00:00)	728dcddf-feaa-11e6-8346-2125e1c055d7	stderr: ./php: /usr/lib64/libcurl.so.4: no version information available (required by ./php)

END RequestId: 728dcddf-feaa-11e6-8346-2125e1c055d7
REPORT RequestId: 728dcddf-feaa-11e6-8346-2125e1c055d7	Duration: 6000.08 ms	Billed Duration: 6000 ms 	Memory Size: 1024 MB	Max Memory Used: 23 MB	
```

In my experience, these errors were often not fatal, but the correct approach is to build the php binary from a base image which is closer to the one lambda uses. So instead of my docker file starting with `FROM ubuntu`, it now starts with `FROM amazonlinux`. Also, with this image, I can use `yum` to install other dependencies like `libpng-devel`. So the new docker build script for producing the php binary looks like this:

```sh dockerfile.buildphp
# Compile PHP with static linked dependencies
# to create a single running binary

FROM amazonlinux

ARG PHP_VERSION

RUN yum install \
    autoconf \
    automake \
    libtool \
    bison \
    re2c \
    libxml2-devel \
    openssl-devel \
    libpng-devel \
    libjpeg-devel \
    curl-devel -y

RUN curl -sL https://github.com/php/php-src/archive/$PHP_VERSION.tar.gz | tar -zxv

WORKDIR /php-src-$PHP_VERSION

RUN ./buildconf --force

RUN ./configure \
    --enable-static=yes \
    --enable-shared=no \
    --disable-all \
    --enable-json \
    --enable-libxml \
    --enable-mbstring \
    --enable-phar \
    --enable-soap \
    --enable-xml \
    --with-curl \
    --with-gd \
    --with-zlib \
    --with-openssl \
    --without-pear

RUN make -j 5
```

If you run this with

    $ sh dockerfile.buildphp

It will use docker to overwrite the php binary which will get shipped when you deploy with `sls deploy`. And this time, there are no more libcurl errors. All the code is [on Github](https://github.com/ZeroSharp/serverless-php).
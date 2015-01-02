# `shibboleth-idp-docker`

## Shibboleth V3.0 Identity Provider Deployment using Docker

This project is a workspace in which I am experimenting with deploying the
[Shibboleth](http://shibboleth.net)
[V3.0 Identity Provider](https://wiki.shibboleth.net/confluence/display/IDP30/Home)
software using the [Docker](http://www.docker.com) container technology.

There's no guarantee that anything here actually works, but if you find
something useful you're welcome to take advantage of it.

## Base Image and Java

Although the Shibboleth IdP *usually* works with the OpenJDK variant of
Java, there have historically been some problems with it and we therefore
recommend the use of Oracle's version.

This Docker build is therefore based on the `oracle-java8` tagged variant of
the [`dockerfile/java`](https://registry.hub.docker.com/u/dockerfile/java/) image.
This is an automated build providing the latest Oracle JDK in an Ubuntu
14.04 (Trusty Tahr) environment. It's quite a large container (about 750MB) but
as I use it for most of my development I don't find it to be significant in practice.

One limitation of this base container is that it doesn't include the
[Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files]
(http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html).
Without these, the IdP can't use some useful encryption algorithms with "long" keys.
One important example is 256-bit AES, which is eligible for use in XML encryption
of messages sent to service providers.

The solution to this is to install the policy files as part of the `docker build`
operation, and the `Dockerfile` assumes you have them available:

* Download `jce_policy-8.zip` [from Oracle](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html).
You will need to accept their license agreement to do this, which is why I don't just include the files here.
* `gunzip jce_policy-8.zip` will generate the `UnlimitedJCEPolicyJDK8`
directory assumed by the `Dockerfile`.

## Fetching Shibboleth and Jetty Distributions

You should execute the `./fetch` script to pull down copies of the Shibboleth IdP distribution and Jetty.
Variables at the top of that file control the version acquired. Some minimal validation is performed of
the downloaded files, but at present it's on a "leap of faith" basis as the key file for Shibboleth is
pulled down from the same server and simply trusted, and Jetty's approach to distribution signing has
been a little hit and miss. Feel free to submit a pull request if you have a better way of handling this.

The distributions are pulled into `fetched/shibboleth-dist` and `fetched/jetty`.

## Shibboleth "Install"

Executing the `./install` script will run the Shibboleth install process. If you do not have a `shibboleth-idp`
directory, this will act like a first-time install and prompt you for critical parameters, resulting in a
basic installation in that directory.

If `shibboleth-idp` already exists, `./install` will act to upgrade it to the latest distribution. This should
be idempotent; you should be able to just run `./install` at any time without changing the results.

Note that because you're running `./install` *outside* the container, you need to have Java set up there as
well as inside, and referenced by the `JAVA_HOME` environmental variable. This is not ideal,
and at some point I may move to performing this operation inside a second container spun up for the purpose.

## Building the Image

Execute the `./build` script to fetch the latest base image and build a new container image. This new image will be
tagged as `shibboleth-idp` and incorporate the Jetty distribution fetched earlier. It will *not* include
the contents of `shibboleth-idp`; instead, they will be mounted into the container at '/opt/shibboleth-idp' when
a container is run from the image.

One important result of this approach is that the container image does not incorporate any secrets that are
part of the Shibboleth configuration, such as passwords. On the other hand, the container image doesn't really
contain much of the IdP, just a tailored environment for it.

## Executing the Container

Start a container from the image using the `./run` script. At the moment, this is set up to be an interactive
container; you will see a couple of lines of logging and then it will appear to pause. Use ^C to stop the
container; it will be automatically removed when you do so.

All state, such as logs, will appear at appropriate locations in the `shibboleth-idp` directory tree.

## OpenSSL Tips

You will often find that you have keys and certificates in a format other than the one you'd like. Here are
some recipes I've found useful in this work.

### Self-signed Certificate and Key to PKCS12

If you have a pair of PEM files (normally a self-signed certificate and the corresponding private key) and
need them in PKCS12 form for use with Jetty, you can try something like this:

    $ openssl pkcs12 -export -out X.p12 -inkey X.key -in X.crt

You'll be prompted for the output password, and the input password if one is required.


## Copyright and License

The entire package is Copyright (C) 2014, Ian A. Young.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


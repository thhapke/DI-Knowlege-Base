# Some Note about Docker on DI 
In general you can use any docker image to run on DI. You only have to ensure that it is correctly tagged so that the pipeline scheduler can select the appropriate docker container that provides the libraries required by the operators. Of course it is a good practice to select a fitting docker image and inherit the setup with $(Path) 

```
FROM $com.sap.opensuse.base
```

But it has a restriction:  You can only use ```pip install``` installations but not the package managers like zypper (opensuse), apt (ununtu), ...

```
FROM com.sap.opensuse.python36

RUN python3 -m pip --no-cache-dir install pandas
```

There is no direct way to open the SAP maintained and quality assured docker containers for modifications using package managers except the one found at: 

/files/vflow/dockerfiles/com/sap/opensuse/golang/zypper

with the definition: 

```
FROM opensuse/leap:15.0

ARG GOPATH=/gopath
ARG GOROOT=/goroot

ENV GOROOT=${GOROOT}
ENV GOPATH=${GOPATH}
ENV PATH=${GOROOT}/bin:${GOPATH}/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

RUN zypper --non-interactive update && \
    # Install tar, gzip, python, python3, pip, pip3, gcc and libgthread
    zypper --non-interactive install --no-recommends --force-resolution \
    tar \
    gzip \
    python=2.7.14 \
    python2-pip \
    python3 \
    python3-pip \
    gcc=7 \
    gcc-c++=7 \
    libgthread-2_0-0=2.54.3 && \
    # Install tornado
    python2 -m pip --no-cache install tornado==5.0.2 && \
    python3 -m pip --no-cache install tornado==5.0.2

COPY sapgolang.tar.gz /tmp/sapgolang.tar.gz

RUN mkdir -p $GOROOT && \
    tar -xzf /tmp/sapgolang.tar.gz --strip-components=1 -C ${GOROOT}

```

This dockerfile can be used for installing additional libraries and tested locally before building this on DI. 

## Reference
[SAP DI Help -  Create Docker](https://help.sap.com/viewer/29ff74dc606c41acad117003f6034ac7/2.6.latest/en-US/62d1df08fa384d0e88bbe9b7cbd2c3fb.html?q=docker)

[SAP DI Help - Docker Inheritance] (https://help.sap.com/viewer/29ff74dc606c41acad117003f6034ac7/2.6.latest/en-US/d49a07c5d66c413ab14731adcfc4f6dd.html)



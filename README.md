# spark-k8
spark with spark driver in k8
# Steps To configure Spark and submit jobs in spark k8 cluster
# 1. Download the respective Spark Jar for version 3.5.2 or the latest
# 2. Unzip the tgz and  `cd spark-3.5.2-bin-hadoop2.7/kubernetes/dockerfiles/spark/Dockerfile`
```
ARG java_image_tag=17-jammy

FROM eclipse-temurin:${java_image_tag}

ARG spark_uid=185

# Before building the docker image, first build and make a Spark distribution following
# the instructions in https://spark.apache.org/docs/latest/building-spark.html.
# If this docker file is being used in the context of building your images from a Spark
# distribution, the docker build command should be invoked from the top level directory
# of the Spark distribution. E.g.:
# docker build -t spark:latest -f kubernetes/dockerfiles/spark/Dockerfile .

RUN set -ex && \
    apt-get update && \
    ln -s /lib /lib64 && \
    apt install -y bash tini libc6 libpam-modules krb5-user libnss3 procps net-tools && \
    mkdir -p /opt/spark && \
    mkdir -p /opt/spark/examples && \
    mkdir -p /opt/spark/work-dir && \
    touch /opt/spark/RELEASE && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/* && rm -rf /var/lib/apt/lists/*

COPY jars /opt/spark/jars
# Copy RELEASE file if exists
COPY RELEAS[E] /opt/spark/RELEASE
COPY bin /opt/spark/bin
COPY sbin /opt/spark/sbin
COPY kubernetes/dockerfiles/spark/entrypoint.sh /opt/
COPY kubernetes/dockerfiles/spark/decom.sh /opt/
COPY examples /opt/spark/examples
COPY kubernetes/tests /opt/spark/tests
COPY data /opt/spark/data

ENV SPARK_HOME /opt/spark

WORKDIR /opt/spark/work-dir
RUN chmod g+w /opt/spark/work-dir
RUN chmod a+x /opt/decom.sh

ENTRYPOINT [ "/opt/entrypoint.sh" ]

# Specify the User that the actual main process will run as
USER ${spark_uid}
```

# 3. Inside from the folder run, `./bin/docker-image-tool.sh -r demo-spark -t 2.4.4 build` this will create a docker image
# 4. Now, create k8 resources: 
> a. Roles
 <img width="596" alt="image" src="https://github.com/user-attachments/assets/b8816601-ebde-4247-b063-0c8e3787406a" />
> b. Service:
<img width="160" alt="image" src="https://github.com/user-attachments/assets/d74ec46a-42cc-48d5-b72d-bbd7901c9e4d" />
> c. ConfigMap:
<img width="351" alt="image" src="https://github.com/user-attachments/assets/eb6cd3ba-86b0-45dc-b779-6a591e0f0d0f" />
> d. Spark-job:
<img width="828" alt="image" src="https://github.com/user-attachments/assets/6c53260f-8909-4ac0-8fb6-590bb0c4fffe" />
<img width="780" alt="image" src="https://github.com/user-attachments/assets/92fb845c-7a9d-45a2-aaf5-9f26273766c4" />
# 5. now apply all the yamls 


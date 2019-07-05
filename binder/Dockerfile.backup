FROM jupyter/scipy-notebook:d4cbf2f80a2a

# copying the repository
COPY . .

# using root
USER root

# ensuring the installation of the notebook package
RUN pip install --no-cache notebook jupyter jupyterlab apt-select
# making building process faster by selecting best mirror
RUN apt-select --country BR && mv sources.list /etc/apt/

# installing apt packages
RUN apt-get update && \
    apt-get install $(grep -vE "^\s*#" ./binder/apt.txt  | tr "\n" " ") -y

# enviroment info
ARG  NB_USER=jovyan
ARG  NB_UID=1000
ENV  USER   ${NB_USER}
ENV  NB_UID ${NB_UID}
ENV  HOME   /home/${NB_USER}

# allowing ssh within the container (for hadoop)
RUN mkdir /var/run/sshd
RUN ssh-keygen -A && chmod 600 /etc/ssh* -R
RUN mkdir .ssh
RUN adduser ${NB_USER} root
RUN adduser ${NB_USER} sudo
RUN echo "${NB_USER} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/user && \
    chmod 0440 /etc/sudoers.d/user

RUN  chown -R ${NB_USER} ${HOME} 

### moving to user setup
USER ${NB_USER}

# binder user requirements
RUN pip install -r ./binder/requirements.txt

# binder user postBuild 
RUN chmod +x ${HOME}/binder/postBuild && \
    sh       ${HOME}/binder/postBuild

### hadoop 

ARG HADOOP_VERSION=hadoop-2.9.2

# hadoop downloading, extracting and removing tar
RUN wget http://ftp.unicamp.br/pub/apache/hadoop/common/${HADOOP_VERSION}/${HADOOP_VERSION}.tar.gz --quiet && \
    tar -xvf ${HADOOP_VERSION}.tar.gz && \
    rm       ${HADOOP_VERSION}.tar.gz

# set env var for java
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 " > .bashrc

# pseudo-distributed operation config files
RUN cp ./resources/hadoop-config/core-site.xml ./${HADOOP_VERSION}/etc/hadoop/ && \
    cp ./resources/hadoop-config/hdfs-site.xml ./${HADOOP_VERSION}/etc/hadoop/ 

# setup passphraseless ssh (forcing the adding to know hosts)
RUN ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa        && \
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
    chmod 0600 ~/.ssh/authorized_keys                    

RUN rm work -R

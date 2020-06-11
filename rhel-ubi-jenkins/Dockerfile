ARG BASE_REGISTRY=registry.access.redhat.com
ARG BASE_IMAGE=ubi8
ARG BASE_TAG=8.1

FROM jenkins/jenkins:2.228-centos as jenkins

FROM ${BASE_REGISTRY}/${BASE_IMAGE}:${BASE_TAG}

LABEL name="Jenkins" \
      description="Jenkins Continuous Integration and Delivery server" \
      vendor="Jenkins" \
      summary="Jenkins 2.228" \
      maintainer="Jenkins" \
      version="2.228"

ENV JAVA_HOME /etc/alternatives/jre_openjdk

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000
ARG JENKINS_HOME=/var/jenkins_home
ARG REF=/usr/share/jenkins/ref

ENV JENKINS_HOME $JENKINS_HOME
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}
ENV REF $REF

COPY --from=jenkins /usr/bin/git-lfs /usr/bin/git-lfs

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN mkdir -p $JENKINS_HOME && \
    chown ${uid}:${gid} $JENKINS_HOME && \
    groupadd -g ${gid} ${group} && \
    useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user} && \
    dnf update -y && \
    dnf install -y git java java-devel unzip --setopt=tsflags=nodocs && \
    git lfs install && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
# $REF (defaults to `/usr/share/jenkins/ref/`) contains all reference configuration we want
# to set on a fresh new installation. Use it to bundle additional plugins
# or config file with your custom jenkins Docker image.
    mkdir -p ${REF}/init.groovy.d && \
    chown -R ${user} "$JENKINS_HOME" "$REF" && \
    chmod 00775 /usr/libexec/openssh/ssh-keysign

# Jenkins home directory is a volume, so configuration and build history
# can be persisted and survive image upgrades
VOLUME $JENKINS_HOME

COPY --from=jenkins /usr/share/jenkins/jenkins.war /usr/share/jenkins/jenkins.war   

ENV JENKINS_UC https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
ENV JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals

# for main web interface:
EXPOSE ${http_port}

# will be used by attached slave agents:
EXPOSE ${agent_port}

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

USER ${user}

COPY --from=jenkins /usr/local/bin/jenkins-support /usr/local/bin/jenkins-support
COPY --from=jenkins /usr/local/bin/jenkins.sh /usr/local/bin/jenkins.sh
COPY --from=jenkins /bin/tini /bin/tini
COPY --from=jenkins /sbin/tini /sbin/tini

ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/jenkins.sh"]

# from a derived Dockerfile, can use `RUN plugins.sh active.txt` to setup $REF/plugins from a support bundle
COPY --from=jenkins /usr/local/bin/plugins.sh /usr/local/bin/plugins.sh
COPY --from=jenkins /usr/local/bin/install-plugins.sh /usr/local/bin/install-plugins.sh

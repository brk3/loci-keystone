FROM openstackloci/requirements as requirements
FROM centos:7

ENV PROJECT=keystone
ARG WHEELS=openstackloci/requirements:centos
ARG PROJECT_REPO=https://git.openstack.org/openstack/${PROJECT}
ARG PROJECT_REF=master
ARG SCRIPTS_REPO=https://github.com/openstack/loci
ARG SCRIPTS_REF=master
ARG OVERRIDE=override

ADD $OVERRIDE /
COPY --from=requirements /root/packages /tmp/packages

RUN set -x \
    && yum upgrade -y \
    && yum install -y git \
        httpd \
        mod_ssl \
        mod_wsgi \
    && git init /tmp/common/ \
    && git --git-dir /tmp/common/.git fetch --depth 1 $SCRIPTS_REPO $SCRIPTS_REF \
    && git --work-tree /tmp/common --git-dir /tmp/common/.git checkout FETCH_HEAD \
    && /tmp/common/scripts/install.sh

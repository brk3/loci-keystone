FROM centos:7

ENV PROJECT=keystone
ARG DOCKER_REPO=yaodu/openstack-requirements
ARG DOCKER_TAG=centos
ARG WHEELS
ARG GIT_REPO=https://github.com/openstack/${PROJECT}
ARG GIT_REF
ARG GIT_REF_REPO=https://git.openstack.org/openstack/${PROJECT}

RUN set -x \
    && yum install --setopt=tsflags=nodocs -y  \
# Project specific packages start
        python \
        httpd \
        mod_ssl \
        mod_wsgi \
# Project specific packages end
    && yum install --setopt=tsflags=nodocs -y git \
# common install start
    && if [ -n "$WHEELS" ]; then \
        curl -sSL ${WHEELS} > /tmp/wheels.tar.gz; \
       else \
        TOKEN=$(curl -sSL "https://auth.docker.io/token?service=registry.docker.io&scope=repository:${DOCKER_REPO}:pull" | \
                    python -c "import sys, json; print json.load(sys.stdin)['token']") \
        && BLOB=$(curl -sSL -H "Authorization: Bearer ${TOKEN}" https://registry.hub.docker.com/v2/${DOCKER_REPO}/manifests/${DOCKER_TAG} | \
                    python -c "import sys, json; print json.load(sys.stdin)['fsLayers'][0]['blobSum']") \
        && curl -sSL -H "Authorization: Bearer ${TOKEN}" https://registry.hub.docker.com/v2/${DOCKER_REPO}/blobs/${BLOB} > /tmp/wheels.tar.gz; \
       fi \
    && git clone ${GIT_REPO} /tmp/${PROJECT} \
    && if [ -n "$GIT_REF" ]; then \
        git --git-dir /tmp/${PROJECT}/.git fetch ${GIT_REF_REPO} ${GIT_REF} \
        && git --git-dir /tmp/${PROJECT}/.git checkout FETCH_HEAD; \
       fi \
    && mkdir /tmp/packages \
    && tar xf /tmp/wheels.tar.gz -C /tmp/packages/ --strip-components=2 root/packages \
    && curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py \
    && python get-pip.py \
    && rm get-pip.py \
    && pip install --no-cache-dir --no-index --no-compile --find-links /tmp/packages --constraint /tmp/packages/upper-constraints.txt /tmp/${PROJECT} \
    && groupadd -g 42424 ${PROJECT} \
    && useradd -u 42424 -g ${PROJECT} -M -d /var/lib/${PROJECT} -s /usr/sbin/nologin -c "${PROJECT} user" ${PROJECT} \
    && mkdir -p /etc/${PROJECT} /var/log/${PROJECT} /var/lib/${PROJECT} /var/cache/${PROJECT} \
    && chown ${PROJECT}:${PROJECT} /etc/${PROJECT} /var/log/${PROJECT} /var/lib/${PROJECT} /var/cache/${PROJECT} \
# common install end
# Project specific command block start
    && pip install --no-cache-dir --no-index --no-compile --find-links /tmp/packages --constraint /tmp/packages/upper-constraints.txt \
        pymysql \
        python-memcached \
        uwsgi \
    && sed -i 's/^Listen 80/#Listen 80/' /etc/httpd/conf/httpd.conf \
    && rm /etc/httpd/conf.d/* \
# Provide compatibility with older format keystone apache config:
    && mkdir -p /var/www/cgi-bin/keystone \
    && cp /bin/keystone-wsgi-public /var/www/cgi-bin/keystone/main \
    && cp /bin/keystone-wsgi-admin /var/www/cgi-bin/keystone/admin \
# Provide compatibility with ubuntu/debian apache:
    && ln -s /usr/sbin/httpd /usr/sbin/apache2 \
    && mkdir -p /etc/apache2/conf-enabled \
    && echo "IncludeOptional /etc/apache2/conf-enabled/*.conf" >> /etc/httpd/conf/httpd.conf \
    && mkdir -p /etc/apache2/mods-available \
    && echo "Include /etc/apache2/mods-available/*.conf" >> /etc/httpd/conf/httpd.conf \
    && touch /etc/apache2/envvars \
# Project specific command block end
    && yum history -y undo $(yum history list git | tail -2 | head -1 | awk '{ print $1}') \
    && rpm -e --nodeps centos-logos \
    && yum clean all \
    && rm -rf /tmp/* /root/.cache \
    && pip uninstall wheel pip -y \
    && find / -type f -name "*.pyc" -delete
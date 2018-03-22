FROM postgres:10

RUN export DEBIAN_FRONTEND=noninteractive \
    && echo 'APT::Install-Recommends "0";\nAPT::Install-Suggests "0";' > /etc/apt/apt.conf.d/01norecommend \
    && apt-get update -y \
    && apt-get upgrade -y \
    && apt-get install -y git curl jq python-psycopg2 python-yaml python-requests python-six python-socks \
        python-dateutil python-pip python-prettytable python-wheel python-psutil python locales \
        openssh-client gcc build-essential cmake \

    ## Make sure we have a en_US.UTF-8 locale available
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \

    && pip install setuptools --upgrade \
    && pip install 'git+https://github.com/soltysh/patroni.git@openshift#egg=patroni[kubernetes]' \

    # build nss_wrapper
    && git clone git://git.samba.org/nss_wrapper.git /tmp/nss_wrapper \
    && mkdir /tmp/nss_wrapper/build \
    && cd /tmp/nss_wrapper/build \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DLIB_SUFFIX=64 .. \
    && make \
    && make install \
    && rm -rf /tmp/nss_wrapper

ENV LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    NSS_WRAPPER_PASSWD=/tmp/passwd \
    NSS_WRAPPER_GROUP=/tmp/group

RUN \
    # create postgresql data directory
    mkdir -p /opt/pgdata \

    # Setup nss wrapper files
    && for path in "$NSS_WRAPPER_PASSWD" "$NSS_WRAPPER_GROUP"; do \
    touch $path && chmod 666 $path ; done \

    # Clean up
    && apt-get remove -y git python-pip python-setuptools git gcc build-essential cmake \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /root/.cache \

    # In order to drop the root user, we have to make some directories world
    # writable as OpenShift default security model is to run the container under
    # random UID.
    && chown -R 1001:0 /opt/pgdata && chmod -R og+rwx /opt/pgdata

ADD entrypoint.sh callback.py nss-wrap.sh /

EXPOSE 5432 8008
USER 1001
ENTRYPOINT ["/nss-wrap.sh"]
CMD ["/entrypoint.sh"]

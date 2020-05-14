FROM python:3.8.2-slim-buster

ENV COMMIT_HASH "48930240573d87d1b06d7504cff41916ce40d28d"

ENV PYTHONUNBUFFERED 1

ENV LIBRARY_PATH /usr/local/lib
ENV LD_LIBRARY_PATH /usr/local/lib
ENV LIBRARY_INCLUDE_PATH /usr/local/include

# PBC
COPY --from=initc3/pbc:0.5.14-slim-buster \
				/usr/local/include/pbc \
				/usr/local/include/pbc
COPY --from=initc3/pbc:0.5.14-slim-buster \
                /usr/local/lib/libpbc.so.1.0.0 \
                /usr/local/lib/libpbc.so.1.0.0
RUN set -ex \
    && cd /usr/local/lib \
    && ln -s libpbc.so.1.0.0 libpbc.so \
    && ln -s libpbc.so.1.0.0 libpbc.so.1

# Setup virtualenv
ENV PYTHON_LIBRARY_PATH /opt/venv
ENV PATH ${PYTHON_LIBRARY_PATH}/bin:${PATH}

# Install charm
# Creates /charm/dist/Charm_Crypto...x86_64.egg, which gets copied into the venv
# /opt/venv/lib/python3.7/site-packages/Charm_crypto...x86_64.egg
RUN set -ex \
        \
        && savedAptMark="$(apt-mark showmanual)" \
        && apt-get update && apt-get install -y --no-install-recommends \
                bison \
                flex \
                gcc \
                git \
                make \
                libc6-dev \
                libgmp-dev \
                libssl-dev \
                wget \
        \
        && mkdir -p /usr/src/charm \
        && git clone https://github.com/JHUISI/charm.git /usr/src/charm \
        && cd /usr/src/charm \
        && git reset --hard $COMMIT_HASH \
        && python -m venv ${PYTHON_LIBRARY_PATH} \
        && ./configure.sh \
        && make install \
        \
        && apt-mark auto '.*' > /dev/null \
	    && apt-mark manual $savedAptMark \
	    && find /usr/local -type f -executable -execdir ldd '{}' ';' \
		        | awk '/=>/ { print $(NF-1) }' \
		        | sort -u \
		        | xargs -r dpkg-query --search \
		        | cut -d: -f1 \
		        | sort -u \
		        | xargs -r apt-mark manual \
	    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
	    && rm -rf /var/lib/apt/lists/* \
        && rm -rf /usr/src/charm

FROM python:3.8.0-buster

ENV COMMIT_HASH "48930240573d87d1b06d7504cff41916ce40d28d"

ENV PYTHONUNBUFFERED 1

RUN apt-get update && apt-get install -y --no-install-recommends \
                bison \
                flex \
        && rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH /usr/local/lib
ENV LD_LIBRARY_PATH /usr/local/lib
ENV LIBRARY_INCLUDE_PATH /usr/local/include

# PBC
COPY --from=sbellem/pbc:0.5.14-buster /usr/local/include/pbc /usr/local/include/pbc
COPY --from=sbellem/pbc:0.5.14-buster /usr/local/lib/libpbc.* /usr/local/lib/

# Setup virtualenv
ENV PYTHON_LIBRARY_PATH /opt/venv
ENV PATH ${PYTHON_LIBRARY_PATH}/bin:${PATH}

# Install charm
# Creates /charm/dist/Charm_Crypto...x86_64.egg, which gets copied into the venv
# /opt/venv/lib/python3.7/site-packages/Charm_crypto...x86_64.egg
RUN set -ex \
        \
        && mkdir -p /usr/src/charm \
        && git clone https://github.com/JHUISI/charm.git /usr/src/charm \
        && cd /usr/src/charm \
        && git reset --hard $COMMIT_HASH \
        && python -m venv ${PYTHON_LIBRARY_PATH} \
        && ./configure.sh \
        && make install \
        && rm -rf /usr/src/charm

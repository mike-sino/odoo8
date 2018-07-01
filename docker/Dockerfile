FROM debian:jessie

LABEL maintainer="mike-sino"
LABEL version="1.0"

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            git \
            sudo \
            curl \
            bash \
            build-essential \
            automake \
            vim \
            node-less \
            node-clean-css \
            python-gevent \
            python-pip \
            python-pyinotify \
            python-renderpm \
            python-support \
        && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb \
        && pip install psycogreen==1.0  \
        && pip install woocommerce

# Install Odoo
ENV ODOO_VERSION 8.0
ENV ODOO_RELEASE 20170815
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        && echo '5835e966a07e5684b4f7bcc39585276b0bb68254 odoo.deb' | sha1sum -c - \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

# Copy entrypoint script and Odoo configuration file
COPY /config/entrypoint.sh /
COPY /config/openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf
RUN chmod +x /entrypoint.sh

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons \
        && chown -R odoo /var/lib/odoo

VOLUME ["/var/lib/odoo", "/mnt/extra-addons", "/usr/lib/python2.7/dist-packages/odoo/addons"]

# Install miniconda3
RUN curl -LO -k https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh  && \
    bash Miniconda3-latest-Linux-x86_64.sh -p /miniconda -b  && \
    rm Miniconda3-latest-Linux-x86_64.sh
ENV PATH $PATH:/miniconda/bin
RUN conda update -y conda  && \
    conda upgrade -y pip && \
    rm -r "/miniconda/pkgs/" && \
    conda clean --all

# Install jupyter lab and extensions
RUN conda install -c conda-forge jupyterLab && \
    mkdir /notebook/ && \
    conda install -c conda-forge nodejs && \
    jupyter labextension install jupyterlab-drawio && \
    jupyter labextension install @jupyterlab/github

# Install python2 and python3 kernels
RUN conda create -n python2 python=2.7 ipykernel && \
    . activate python2 && \
    python -m ipykernel install --user --name=python2 --display-name=python2 && \
    . deactivate

# Expose Odoo services
EXPOSE 8069 8080 9001

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/sbin/init"]
FROM databricksruntime/minimal:latest

RUN apt-get update \
  && apt-get install -y openssh-server \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR /databricks
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -q -y --force-yes --fix-missing --ignore-missing \
        ganglia-monitor \
        ganglia-webfrontend \
        ganglia-monitor-python \
        python-pip \
        rsync \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Upgrade Ganglia to 3.7.2 to patch XSS bug, see CJ-15250
# Upgrade Ganglia to 3.7.4 and use private forked repo to patch several security bugs, see CJ-20114
# SC-17279: We run gmetad as user ganglia, so change the owner from nobody to ganglia for the rrd directory
RUN cd /tmp \
  && export GANGLIA_WEB=ganglia-web-3.7.4-db-4 \
  && wget https://s3-us-west-2.amazonaws.com/databricks-build-files/$GANGLIA_WEB.tar.gz \
  && tar xvzf $GANGLIA_WEB.tar.gz \
  && cd $GANGLIA_WEB \
  && make install \
  && chown ganglia:ganglia /var/lib/ganglia/rrds
# Install Phantom.JS
RUN cd /tmp \
  && export PHANTOM_JS="phantomjs-2.1.1-linux-x86_64" \
  && wget https://s3-us-west-2.amazonaws.com/databricks-build-files/$PHANTOM_JS.tar.bz2 \
  && tar xvjf $PHANTOM_JS.tar.bz2 \
  && mv $PHANTOM_JS /usr/local/share \
  && ln -sf /usr/local/share/$PHANTOM_JS/bin/phantomjs /usr/local/bin
# Apache2 config. The `sites-enabled` config files are loaded into the container
# later.
RUN rm /etc/apache2/sites-enabled/* && a2enmod proxy && a2enmod proxy_http
# Install Ganglia API server & support for Ganglia Python modules
# Remove all modules besides the nvidia.pyconf module (used for GPU metrics)
# Note, we originally use "pip install -r ganglia-api/requirements.txt" to install ganglia-api's
# dependency, which is tornado. However, this requirement file (https://github.com/guardian/ganglia-api/blob/master/requirements.txt)
# does not specify the version and tornado will break PythonShell.py when it imports
# from IPython.nbconvert.filters.ansi import ansi2html. It is probably caused by the fact that
# IPython's version is too old. For now, we just pin the version of tornado to the version used by
# Runtime 5.2. Once we move to Runtime 6, we should be able to use a newer version of tornado
# because PythonShell should not depend on nbconvert.
RUN export GANGLIA_API=ganglia-api-cbe773d051168e05118774708ff7a0ce881617f4 \
  && wget https://s3-us-west-2.amazonaws.com/databricks-build-files/$GANGLIA_API.tar.gz \
  && tar xvzf $GANGLIA_API.tar.gz \
  && pip install tornado==5.1.1 \
  && apt-get install ganglia-monitor-python \
  && find /etc/ganglia/conf.d/*.pyconf ! -name 'nvidia.pyconf' -type f -exec rm -f {} + \
  && find /etc/ganglia/conf.d/*.pyconf.disabled ! -name 'nvidia.pyconf' -type f -exec rm -f {} + \
  && find /usr/lib/ganglia/python_modules/*.py ! -name 'nvidia.py' -type f -exec rm -f {} +
# Stop Ganglia related services from starting on boot
RUN update-rc.d ganglia-monitor disable \
  && update-rc.d gmetad disable \
  && update-rc.d apache2 disable


FROM databricksruntime/standard:latest

RUN apt-get update \
  && apt-get install -y \
	curl \
	make \
	cmake \
	libtool \
	m4 \
	automake \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN curl -L https://github.com/tsl0922/ttyd/archive/refs/tags/1.6.3.tar.gz | tar zx && \
   mv ttyd-* ttyd && \
   cd /ttyd && BUILD_TARGET=x86_64 ./scripts/cross-build.sh && \
   mkdir -p /databricks/driver/logs && \
   mkdir -p /databricks/spark/scripts/ttyd/ && \
   mv /ttyd/build/ttyd /databricks/spark/scripts/ttyd/

ENV TTYD_DIR=/databricks/spark/scripts/ttyd/
ENV TTYD_BIN_FILE=/databricks/spark/scripts/ttyd/ttyd
   
COPY scripts/setup_ttyd_daemon.sh $TTYD_DIR/setup_ttyd_daemon.sh
COPY scripts/stop_ttyd_daemon.sh $TTYD_DIR/stop_ttyd_daemon.sh
COPY scripts/start_ttyd_daemon.sh $TTYD_DIR/start_ttyd_daemon.sh
COPY scripts/webTerminalBashrc $TTYD_DIR/webTerminalBashrc
COPY scripts/monit /etc/init.d/monit

RUN mkdir -p /etc/monit/conf.d/ 
COPY scripts/ttyd-daemon-not-active /etc/monit/conf.d/ttyd-daemon-not-active



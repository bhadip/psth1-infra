FROM lscr.io/linuxserver/webtop:debian-openbox

RUN dpkg --add-architecture i386 && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        wine wine32:i386 wine64 winetricks tmux && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

COPY mt4-service/run /etc/services.d/mt4/run
RUN chmod +x /etc/services.d/mt4/run

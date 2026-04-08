FROM homeassistant/home-assistant:2026.2

COPY docker/start.sh /tmp/start.sh

RUN chmod +x /tmp/start.sh

COPY config /config

CMD ["bash","/tmp/start.sh"]

FROM homeassistant/home-assistant:2026.1

COPY docker/start.sh /tmp/start.sh

RUN chmod +x /tmp/start.sh

COPY config /config

CMD ["bash","/tmp/start.sh"]

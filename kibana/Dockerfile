# https://github.com/elastic/kibana-docker
FROM docker.elastic.co/kibana/kibana:6.3.2

# Add your kibana plugins setup here
# Example: RUN kibana-plugin install <name|url>

# Sentinl
# COPY sentinl-v6.3.2.zip /usr/share/kibana
# RUN kibana-plugin install file:///usr/share/kibana/sentinl-v6.3.2.zip
RUN kibana-plugin install https://github.com/sirensolutions/sentinl/releases/download/tag-6.4.2-0/sentinl-v6.3.2.zip

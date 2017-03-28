FROM nginx

ENV CT_URL https://releases.hashicorp.com/consul-template/0.18.1/consul-template_0.18.1_linux_amd64.tgz
ENV CT_FILE /etc/consul-templates/nginx.conf
ENV CT_LOG_LEVEL debug

ENV NX_FILE /etc/nginx/conf.d/app.conf

ENV CONSUL consul:8500
ENV SERVICE consul-8500

RUN apt-get update && apt-get install -y --no-install-recommends curl \
 && curl -sSL $CT_URL | tar xvzC /usr/local/bin \
 && rm -v /etc/nginx/conf.d/* \
 && curl -sSL http://stedolan.github.io/jq/download/linux64/jq > /usr/local/bin/jq \
 && chmod u+x /usr/local/bin/jq \
 && mkdir /etc/consul-templates

# Command will
# 1. Write Consul Template File
# 2. Start Nginx
# 3. Start Consul Template

CMD echo "upstream $SERVICE {                                            \n\
     least_conn;                                                         \n\
     {{range service \"$SERVICE\" }}                                     \n\
     server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1; \n\
     {{else}}server 127.0.0.1:65535; {{end}}                             \n\
   }                                                                     \n\
 server {                                                                \n\
   listen 80 default_server;                                             \n\
   location / {                                                          \n\
     proxy_pass http://$SERVICE;                                         \n\
     proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;       \n\
     proxy_set_header Host \$host;                                       \n\
     proxy_set_header X-Real-IP \$remote_addr;                           \n\
   }                                                                     \n\
 }" > $CT_FILE; \
 /usr/sbin/nginx -c /etc/nginx/nginx.conf \
 & CONSUL_TEMPLATE_LOG=$CT_LOG_LEVEL /usr/local/bin/consul-template \
   -consul-addr=$CONSUL \
   -template "$CT_FILE:$NX_FILE:/usr/sbin/nginx -s reload";

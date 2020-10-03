ROM alpine:latest

# install acf + deps
RUN apk update \
    && apk upgrade \
    && apk add alpine-conf acf-core acf-alpine-baselayout acf-openssl

# Create volume folders.
RUN mkdir -p /volume/acf
RUN mkdir -p /volume/cert
RUN mkdir -p /volume/certs
RUN mkdir -p /volume/mini_httpd
RUN mkdir -p /volume/private
RUN mkdir -p /volume/req
RUN mkdir -p /volume/ssl

# Store acf completely in volume
RUN mv    /etc/acf /volume/
RUN ln -s -n /volume/acf  /etc/acf

# setup acf (blocks root account, we create new passwd file later)
RUN setup-acf

# set initial acf passwd
# root:alpine
RUN echo "root:$(mkpasswd -m sha256 alpine):Admin account:ADMIN" > /etc/acf/passwd

# Not using openrc
RUN poweroff
RUN rc-update del mini_httpd default

# Change $dir location to volume.
RUN sed -i -r 's:dir\s+=\s+/etc/ssl:dir = /volume:g' /etc/ssl/openssl-ca-acf.cnf

# Remove empty generated certificate and request directory (goes to volume top level)
RUN rm -rf /etc/ssl/cert
RUN rm -rf /etc/ssl/req

# Move httpd settings, cert
RUN mv /etc/mini_httpd/mini_httpd.conf    /volume/mini_httpd/
RUN mv /etc/ssl/mini_httpd/server.pem     /volume/mini_httpd/

# Move complete ssl directory to volume
RUN mv /etc/ssl /volume
RUN ln -s -n /volume/ssl /etc/ssl

# Link certificate and request directory, also link mini_httpd conf
RUN ln -s -n /volume/cert /volume/ssl/cert
RUN ln -s -n /volume/req  /volume/ssl/req
RUN ln -s -n /volume/mini_httpd/mini_httpd.conf /etc/mini_httpd/mini_httpd.conf
RUN ln -s -n /volume/mini_httpd/server.pem      /volume/ssl/mini_httpd/server.pem

# Export
VOLUME /volume
EXPOSE 443
ENTRYPOINT ["/usr/sbin/mini_httpd", "-D", "-C", "/etc/mini_httpd/mini_httpd.conf"]

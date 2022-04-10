FROM alpine:latest
LABEL maintainer="Brian Moore"

# Install Python for Ansible
ENV PYTHONUNBUFFERED=1
RUN apk add --update --no-cache python3 && ln -sf python3 /usr/bin/python
RUN python3 -m ensurepip
RUN pip3 install --no-cache --upgrade pip setuptools

# Install SSH
RUN apk add --update --no-cache openssh
RUN echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config

# Setup User
RUN adduser -h /home/testuser -s /bin/sh -D testuser
RUN echo -n 'testuser:testpassword' | chpasswd

# Set Entrypoint and port
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 22
COPY entrypoint.sh /

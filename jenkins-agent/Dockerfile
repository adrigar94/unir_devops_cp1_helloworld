FROM jenkins/jenkins:lts-jdk21

USER root

RUN apt-get update && apt-get install -y \
    openssh-server python3 python3-pip  default-jre\
    sudo \
    && rm -rf /var/lib/apt/lists/*

# Instala paquetes Python globalmente con override de protección
RUN pip3 install --break-system-packages --upgrade pip && \
    pip3 install --break-system-packages pytest flask

RUN mkdir /var/run/sshd

# Establece la contraseña 'jenkinspassword' para el usuario 'jenkins'
RUN echo "jenkins:jenkinspassword" | chpasswd && \
    echo "jenkins ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

EXPOSE 22

ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"

COPY wiremock /opt/wiremock
RUN chmod -R a+r /opt/wiremock

CMD ["/usr/sbin/sshd", "-D"]

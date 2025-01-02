# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:18.04

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="Antón López Álvarez <antlopez@birt.eus>"

# Actualizamos la lista de paquetes e instalamos todos los paquetes necesarios
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 \
    proftpd openssl ssh git && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero \
    /var/www/html/sitiosegundo \
    /root/.ssh /srv/ftp/proyecto

# Copiamos archivos al contenedor
COPY apache/indexPrimero.html /var/www/html/sitioprimero/index.html
COPY apache/sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf
COPY apache/sitioPrimero.key /etc/ssl/private/sitioPrimero.key
COPY apache/SitioPrimero.cer /etc/ssl/certs/SitioPrimero.cer

COPY apache/indexSegundo.html /var/www/html/sitiosegundo/index.html
COPY apache/sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf

COPY ftp/proftpd.conf /etc/proftpd/
COPY ftp/proftpd.crt.pem /etc/ssl/certs/
COPY ftp/proftpd.key.pem /etc/ssl/private/
COPY ftp/tls.conf /etc/proftpd/

COPY ssh/sshd_config /etc/ssh/sshd_config
COPY ssh/antlopez2.pub /usr/local/authorized_keys

COPY ssh/claverepoanton /root/.ssh/id_rsa

# Se crean los usuarios con sus rutas home y se veta al segundo usuario del FTP
RUN useradd antlopez1 --password $(openssl passwd -1 deaw) \
    --home /var/www/html/sitioprimero/ && \
    useradd antlopez2 --password $(openssl passwd -1 deaw) \
    --home /var/www/html/sitiosegundo/ && \
    echo "antlopez2" >> /etc/ftpusers

# Se otorgan permisos para poder navegar por los ficheros HTML
RUN chmod -R 755 /var/www/html/sitioprimero /var/www/html/sitiosegundo /srv/ftp /usr/local/authorized_keys
#RUN chmod 644 /var/www/html/sitioprimero/* /var/www/html/sitiosegundo/*
RUN chown -R antlopez1:antlopez1 /var/www/html/sitioprimero && \
    chown -R antlopez2:antlopez2 /var/www/html/sitiosegundo

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

#RUN chmod 400 /root/.ssh/claverepoanton
RUN chmod 400 /root/.ssh/id_rsa

# Añade la clave SSH al agente y clona el repositorio
RUN eval "$(ssh-agent -s)" && \
    ssh-add /root/.ssh/id_rsa && \
#    ssh-add /root/.ssh/id_rsa && \
    ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts && \
#    git clone git@github.com:deaw-birt/UD3-Dockerfile.git /srv/ftp/
    git clone git@github.com:antlopezbirt/prueba.git /srv/ftp/proyecto/

RUN chown -R ftp:nogroup /srv/ftp

# Exponemos los puertos
EXPOSE 21 33 50000-50030 80 443

# Comando por defecto al iniciar el contenedor
CMD ["/bin/bash", "-c", "proftpd ; service ssh start ; apachectl -D FOREGROUND"]

#
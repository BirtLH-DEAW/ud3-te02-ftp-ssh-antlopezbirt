# ETAPA 1/2: Clona el repositorio en una imagen alpine/git
FROM alpine/git AS inicial

# Adquiere la clave privada
COPY --chmod=400 ssh/clave_deaw /root/.ssh/id_rsa

# Añade Github.com a los hosts conocidos de SSH y clona
RUN \
    ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
    && git clone git@github.com:deaw-birt/UD3-Dockerfile.git /srv/ftp/

# ETAPA 2/2: Genera la imagen final donde se copiará el repositorio clonado (desaparecerá el rastro de la clave privada)
FROM ubuntu:18.04

# Establece los metadatos de la imagen
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="Anton Lopez Alvarez <antlopez@birt.eus>"

# Actualiza e instala todos los paquetes necesarios, en esta ultima fase no se instala git, ya no se necesita
RUN \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 \
    proftpd openssl ssh && \
    rm -rf /var/lib/apt/lists/*

# Copia todos los recursos al contenedor
COPY \
    apache/indexPrimero.html apache/sitioPrimero.conf apache/sitioPrimero.key \
    apache/SitioPrimero.cer apache/indexSegundo.html apache/sitioSegundo.conf \
    ftp/proftpd.conf ftp/proftpd.crt.pem ftp/proftpd.key.pem ftp/tls.conf \
    ssh/sshd_config ssh/antlopez2.pub /

RUN \
    # Crea los directorios para alojar recursos y configuraciones
    mkdir -p    /var/www/html/sitioprimero \
                /var/www/html/sitiosegundo \
                /root/.ssh /srv/ftp/proyecto && \
    #Crea el grupo apacheFTP e incluye en él a Apache y a proFTPd
    groupadd apacheFTP && \
    usermod -G apacheFTP www-data && \
    usermod -G apacheFTP ftp && \
    # Se crean los usuarios con sus rutas home y se veta al segundo usuario del FTP
    useradd antlopez1 --password $(openssl passwd -1 deaw) \
        --home /var/www/html/sitioprimero/ --shell /usr/sbin/nologin && \
    useradd antlopez2 --password $(openssl passwd -1 deaw) \
        --home /var/www/html/sitiosegundo/ && \
    echo "antlopez2" >> /etc/ftpusers 

RUN \
    #Mueve los ficheros copiados a su ubicación correspondiente
    mv indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf && \
    mv sitioPrimero.key /etc/ssl/private/sitioPrimero.key && \
    mv SitioPrimero.cer /etc/ssl/certs/SitioPrimero.cer && \
    mv indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf && \
    mv proftpd.conf /etc/proftpd/ && \
    mv proftpd.crt.pem /etc/ssl/certs/ && \
    mv proftpd.key.pem /etc/ssl/private/ && \
    mv tls.conf /etc/proftpd/ && \
    mv sshd_config /etc/ssh/sshd_config && \
    mv antlopez2.pub /usr/local/authorized_keys

RUN \
    # Cambia los permisos de la clave pública
    chmod 755 /usr/local/authorized_keys && \
    # Cambia la propiedad de grupo de los directorios a utilizar por los usuarios
    chown -R :apacheFTP /var/www/html && \
    # Establece el permiso setGID para que el grupo apacheFTP pueda acceder a los ficheros subidos por FTP o SSH
    chmod -R 2755 /var/www/html && \
    # Establece el usuario propietario de cada directorio a utilizar
    chown -R antlopez1 /var/www/html/sitioprimero && \
    chown -R antlopez2 /var/www/html/sitiosegundo && \
    # Habilita los sitios y el módulo SSL
    a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# Copia el repositorio clonado en la fase inicial
COPY --from=inicial /srv/ftp /srv/ftp/proyecto/

# Expone los puertos a publicar. El 33 para SSH y el rango 50000-50030 para FTP pasivo
EXPOSE 21 33 50000-50030 80 443

# Al arrancar el contenedor se inician proftpd, ssh y Apache
CMD ["/bin/bash", "-c", "proftpd ; service ssh start ; apachectl -D FOREGROUND"]
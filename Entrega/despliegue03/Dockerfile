# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:latest

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="mbarrientos <mbarrientos@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano, apache2, openssl, git y proftp
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd proftpd-mod-crypto openssh-server git && \
    rm -rf /var/lib/apt/lists/*

# Copiamos archivos al contenedor
# archivos web, apache y certificado
COPY /html /var/www/html
COPY /apache/sitioPrimero.conf /etc/apache2/sites-available
COPY /apache/sitioSegundo.conf /etc/apache2/sites-available
COPY /ssl/sitioPrimero.key /etc/ssl/private
COPY /ssl/SitioPrimero.cer /etc/ssl/certs

# archivos ftp
COPY /ftp/proftpd.conf /etc/proftpd/
COPY /ftp/tls.conf /etc/proftpd/
COPY /ssl/proftpd.crt /etc/ssl/certs/
COPY /ssl/proftpd.key /etc/ssl/private/
COPY /ftp/modules.conf /etc/proftpd/
COPY /ftp/ftpusers /etc/

# archivos ssh
COPY /ssh/github_key /etc/ssh
COPY /ssh/github_key.pub /etc/ssh
COPY /ssh/ssh_config /etc/ssh
COPY /ssh/sshd_config /etc/ssh

# creamos carpeta donde alojar el repositorio github
RUN mkdir -p /var/www/html/git-ssh
# permisos para leer la llave
RUN chmod 600 /etc/ssh/github_key
# clonamos repositorio github
RUN GIT_SSH_COMMAND="ssh -i /etc/ssh/github_key -o StrictHostKeyChecking=no" \
    git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /var/www/html/git-ssh

# creamos los usuarios martin1 martin2 y martin
# martin1 no podrá logearse en el sistema
RUN useradd -d /var/www/html/sitioprimero -s /bin/false martin1 && echo "martin1:martin" | chpasswd
# los demás si
RUN useradd -d /var/www/html/sitiosegundo -s /bin/bash martin2 && echo "martin2:martin" |chpasswd
RUN chown martin2:martin2 /var/www/html/sitiosegundo
RUN useradd -m -s /bin/bash martin 

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21
EXPOSE 990
EXPOSE 50000-50030
EXPOSE 2222

# Comando por defecto al iniciar el contenedor
CMD service apache2 start && service proftpd start && service ssh start && tail -f /dev/null
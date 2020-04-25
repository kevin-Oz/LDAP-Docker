#Configuracion de  LDAP-MASTER
FROM debian:bullseye
RUN apt-get update && echo 'slapd/root_password password password' | debconf-set-selections &&\
    echo 'slapd/root_password_again password password' | debconf-set-selections && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y slapd ldap-utils
## configurar slapd
RUN echo "slapd slapd/no_configuration boolean false" | debconf-set-selections \
&& echo "slapd slapd/domain string paneschucos.occ.ues.edu.sv" | debconf-set-selections \
&& echo "slapd shared/organization string 'paneschucos'" | debconf-set-selections \
&& echo "slapd slapd/password1 password admin" | debconf-set-selections \
&& echo "slapd slapd/password2 password admin" | debconf-set-selections \
&& echo "jslapd slapd/backend select HDB" | debconf-set-selections \
&& echo "slapd slapd/purge_database boolean true" | debconf-set-selections \
&& echo "slapd slapd/allow_ldap_v2 boolean false" | debconf-set-selections \
&& echo "slapd slapd/move_old_database boolean true" | debconf-set-selections 

RUN dpkg-reconfigure -f noninteractive slapd
#crear archivos de config para replicacion y usuarios .ldif
WORKDIR /
RUN echo "\
dn: olcOverlay=syncprov,olcDatabase={1}mdb,cn=config\n\
objectClass: olcOverlayConfig\n\
objectClass: olcSyncProvConfig\n\
olcOverlay: syncprov\n\
olcSpSessionLog: 100\n" >> syncprov.ldif \
&& echo "\
dn: cn=module,cn=config\n\
objectClass: olcModuleList\n\
cn: module\n\
olcModulePath: /usr/lib/ldap\n\
olcModuleLoad: syncprov.la" >> mod_syncprov.ldif \
&& echo "\
dn: ou=usuarios,dc=paneschucos,dc=occ,dc=ues,dc=edu,dc=sv\n\
objectClass: organizationalUnit\n\
objectClass: top\n\
ou: usuarios " >> organizacion.ldif
## falta crear archivo para usuario lector para no usar admin como user de replicacion

#exponer puerto de ldap
EXPOSE 389

RUN echo "#\!/bin/bash \n\
service slapd start  && ldapadd -Y EXTERNAL -H ldapi:/// -f mod_syncprov.ldif && ldapadd -Y EXTERNAL -H ldapi:/// -f syncprov.ldif && \
tail -f /dev/null " > entrypoint.sh && chmod +x entrypoint.sh
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]


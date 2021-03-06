# (C) Copyright FEI Systems Inc.
#

FROM ubuntu:latest

MAINTAINER John Dondapati <john.dondapati@feisystems.com> (@johndondapati)

RUN apt-get update && apt-get dist-upgrade -y  \
    && apt-get install -y wget unzip \
    && apt-get autoclean && apt-get --purge -y autoremove \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install JRE
ENV JAVA_VER=8 \
    JAVA_REL=0 \
    JAVA_MF=2.10
RUN TARGET_ARCH=$(uname -i) \
    && JAVA_VRMF=$JAVA_VER.$JAVA_REL-$JAVA_MF \
    && YML_FILENAME=ibm-java-jre-$JAVA_VRMF-linux-$TARGET_ARCH-javase.yml \
    && BASE_URL="https://public.dhe.ibm.com/ibmdl/export/pub/systems/cloud/runtimes/java/meta" \
    && wget -q -U UA-IBM_JAVA_Docker -O /tmp/ibm-java.yml $BASE_URL/$JAVA_VER$JAVA_REL/$YML_FILENAME \
    && JAVA_URL=$(cat /tmp/ibm-java.yml | sed -n 's/\s*uri:\s//p' | tr -d '\r') \
    && wget -q -U UA-IBM-JAVA-Docker -O /tmp/ibm-java.bin $JAVA_URL \
    && ESUM=$(cat /tmp/ibm-java.yml | sed -n 's/\s*md5sum:\s//p' | tr -d '\r') \
    && echo "$ESUM /tmp/ibm-java.bin" | md5sum -c - \
    && rm -f /tmp/ibm-java.yml \
    && echo "INSTALLER_UI=silent" > /tmp/response.properties \
    && echo "USER_INSTALL_DIR=/opt/ibm/java" >> /tmp/response.properties \
    && echo "LICENSE_ACCEPTED=TRUE" >> /tmp/response.properties \
    && mkdir -p /opt/ibm \
    && chmod +x /tmp/ibm-java.bin \
    && /tmp/ibm-java.bin -i silent -f /tmp/response.properties \
    && rm -f /tmp/response.properties \
    && rm -f /tmp/ibm-java.bin
ENV JAVA_HOME=/opt/ibm/java \
    PATH=/opt/ibm/java/jre/bin:$PATH

# Install WebSphere Liberty
ENV LIBERTY_VERSION 8.5.5_08
RUN LIBERTY_URL=$(wget -q -O - https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/wasdev/downloads/wlp/index.yml  | grep $LIBERTY_VERSION -A 6 | sed -n 's/\s*kernel:\s//p' | tr -d '\r')  \
    && wget -q $LIBERTY_URL -U UA-IBM-WebSphere-Liberty-Docker -O /tmp/wlp.zip \
    && unzip -q /tmp/wlp.zip -d /opt/ibm \
    && rm /tmp/wlp.zip 	
ENV PATH=/opt/ibm/wlp/bin:$PATH

# Set Path Shortcuts
ENV LOG_DIR=/logs \
    WLP_OUTPUT_DIR=/opt/ibm/wlp/output
RUN ln -s $WLP_OUTPUT_DIR/defaultServer /output \
    && ln -s /opt/ibm/wlp/usr/servers/defaultServer /config

# Configure WebSphere Liberty
RUN /opt/ibm/wlp/bin/server create \
    && rm -rf $WLP_OUTPUT_DIR/.classCache
EXPOSE 9080 9443

#making liberty common profile

RUN installUtility install --acceptLicense collectiveMember-1.0 monitor-1.0 webCache-1.0 ldapRegistry-3.0 appSecurity-2.0 localConnector-1.0 restConnector-1.0 ssl-1.0 requestTiming-1.0 sessionDatabase-1.0 


#Making the liberty webprofile7

COPY server.xml /opt/ibm/wlp/usr/servers/defaultServer/ 
RUN installUtility install --acceptLicense defaultServer \
    && rm -rf /opt/ibm/wlp/usr/servers/defaultServer/workarea
	
CMD ["/opt/ibm/wlp/bin/server", "run", "defaultServer"]

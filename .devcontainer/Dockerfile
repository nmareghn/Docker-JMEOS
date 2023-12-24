FROM debian:bookworm-slim
MAINTAINER nidhalmareghni8@gmail.com

# install corretto after verifying that the key is the one we expect.
RUN sudo apt-get -y update \
  && sudo apt-get install -y git curl gnupg build-essential tree vim cmake postgresql-server-dev-15 libproj-dev libjson-c-dev libgsl-dev libgeos-dev postgis \
  && sudo export GNUPGHOME="$(mktemp -d)" \
  && sudo curl -fL https://apt.corretto.aws/corretto.key | gpg --batch --import \
  && sudo gpg --batch --export '6DC3636DAE534049C8B94623A122542AB04F24E3' > /usr/share/keyrings/corretto.gpg \
  && sudo rm -r "$GNUPGHOME" \
  && sudo unset GNUPGHOME \
  && sudo echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" > /etc/apt/sources.list.d/corretto.list \
  && sudo apt-get update \
  && sudo apt-get install -y java-21-amazon-corretto-jdk 

# BUILD MobilityDB
RUN sudo git clone https://github.com/MobilityDB/MobilityDB.git -b develop /usr/local/src/MobilityDB
RUN sudo mkdir -p /usr/local/src/MobilityDB/build 
RUN sudo cd /usr/local/src/MobilityDB/build && \
    cmake -DMEOS=ON .. && \
    make -j$(nproc) && \
    make install

# COMMON FOR ALL IMAGES
ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "/root/.m2"

COPY --from=maven:3.9.6-eclipse-temurin-11 ${MAVEN_HOME} ${MAVEN_HOME}
COPY --from=maven:3.9.6-eclipse-temurin-11 /usr/local/bin/mvn-entrypoint.sh /usr/local/bin/mvn-entrypoint.sh
COPY --from=maven:3.9.6-eclipse-temurin-11 /usr/share/maven/ref/settings-docker.xml /usr/share/maven/ref/settings-docker.xml

RUN ln -s ${MAVEN_HOME}/bin/mvn /usr/bin/mvn

# ADD PROJECT MobilityDB-JMEOS
RUN sudo git clone https://github.com/nmareghn/MobilityDB-JMEOS  /usr/local/jmeos
RUN sudo rm /usr/local/jmeos/src/main/resources/lib/libmeos.so
RUN sudo cp /usr/local/lib/libmeos.so /usr/local/jmeos/src/main/resources/lib/libmeos.so
RUN sudo rm /usr/local/jmeos/jar/*

# CLEAN_UP
RUN sudo rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*lists/* 
RUN sudo apt-get remove --purge --autoremove -y curl gnupg 

# END_POINT
WORKDIR /usr/local/jmeos
CMD ["/bin/bash"]



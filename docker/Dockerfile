ARG IMAGE_TAG=11-al2-full
ARG BASE_IMAGE=public.ecr.aws/ews-network/amazoncorretto
ARG BUILD_IMAGE=$BASE_IMAGE:$IMAGE_TAG

FROM $BUILD_IMAGE as builder
ENV M2_HOME=/opt/apache-maven-3.8.1
ENV MAVEN_HOME=/opt/apache-maven-3.8.1
ENV PATH=${M2_HOME}/bin:${PATH}
ENV JAVA_HOME=/usr/lib/jvm/java-11-amazon-corretto.x86_64/
ADD https://dlcdn.apache.org/maven/maven-3/3.8.3/binaries/apache-maven-3.8.3-bin.tar.gz /opt/apache-maven.tar.gz
ADD https://www.scala-sbt.org/sbt-rpm.repo /etc/yum.repos.d/sbt-rpm.repo
RUN (tar --version 2>/dev/null || yum install tar gzip -y);\
    tar -zxf /opt/apache-maven.tar.gz -C /opt ; \
    mvn --version; \
    yum install sbt -y
COPY src /opt/src
COPY project /opt/project
COPY build.sbt /opt/build.sbt
COPY examples /opt/examples
WORKDIR /opt
ENV JAR_FILE $JAR_FILE
RUN sbt clean compile pack -DskipTests=true
RUN find target -name "*.jar" -print
RUN find target -name "*kafka-lag-exporter*.jar" -type f -print -exec cp {} /opt/app.jar \;


FROM $BASE_IMAGE:11

RUN yum upgrade -y --exclude=*devel ;                                                            \
    yum install -y shadow-utils ;                                               \
    jq --version || yum install jq -y ;                                         \
    groupadd -r app -g 1042 &&                                                  \
    useradd -u 1042 -r -g app -m -d /app -s /sbin/nologin -c "App user" app &&  \
    chown -R app: /app &&                                                       \
    yum erase shadow-utils -y && yum clean all && rm -rfv /var/cache/yum

COPY --chown=app:app --from=builder //opt/target/pack /app/
COPY --chown=app:app --from=builder //opt/examples/standalone/logback.xml /app/logback.xml
WORKDIR /app
ENV PATH=/app/bin:$PATH
USER app
ENTRYPOINT ["kafka-lag-exporter"]
CMD ["-Dlogback.configurationFile=/app/logback.xml"]

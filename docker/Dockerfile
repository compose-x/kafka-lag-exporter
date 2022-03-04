ARG IMAGE_TAG=11
ARG BASE_IMAGE=public.ecr.aws/ews-network/amazoncorretto

FROM ${BASE_IMAGE}:${IMAGE_TAG}
ARG APP_BUILD

RUN yum upgrade -y --exclude=*devel ;                                           \
    yum install -y shadow-utils ;                                               \
    groupadd -r app -g 1042 &&                                                  \
    useradd -u 1042 -r -g app -m -d /app -s /sbin/nologin -c "App user" app &&  \
    chown -R app: /app &&                                                       \
    yum erase shadow-utils -y && yum clean all && rm -rfv /var/cache/yum

COPY $APP_BUILD /app/
COPY examples/standalone/logback.xml /app/logback.xml
WORKDIR /app
ENV PATH=/app/bin:$PATH
USER app
ENTRYPOINT ["kafka-lag-exporter"]
CMD ["-Dlogback.configurationFile=/app/logback.xml"]

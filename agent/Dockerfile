FROM public.ecr.aws/appmesh/aws-appmesh-envoy:v1.27.2.0-prod as envoy

FROM public.ecr.aws/amazonlinux/amazonlinux:2

RUN yum -y update && \
    yum -y install procps htop unzip less && \
    # Install aws cli
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm awscliv2.zip && \
    yum clean all && \
    rm -rf /var/cache/yum

COPY --from=envoy /aws_appmesh_aggregate_stats.wasm /aws_appmesh_aggregate_stats.wasm
COPY --from=envoy /health_check.sh /health_check.sh

# Copy from local
ADD agent /usr/bin/agent
ADD envoy /usr/bin/envoy
ADD resources /agent-resources

RUN chmod +x /usr/bin/envoy \
  /usr/bin/agent
RUN chmod -R 0755 /agent-resources
RUN mkdir -p /tmp && chmod 4777 /tmp

# https://man7.org/linux/man-pages/man7/capabilities.7.html
# When AppNet agent is run as a non-root user, linux capabilities are not preserved on the Envoy process
# These capabilities are required for tproxy settings in Envoy for service connect bridge mode
RUN setcap CAP_SETPCAP,CAP_NET_ADMIN+p /usr/bin/agent

CMD [ "/usr/bin/agent" ]

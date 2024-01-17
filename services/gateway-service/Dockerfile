FROM golang:1.17-buster as builder

WORKDIR /app

COPY go.* ./
RUN go mod download

COPY . ./

# Build the binary.
# build app with seeker agent
ENV SEEKER_SERVER_URL=https://poc271.seeker.synopsys.com:443
ENV SEEKER_PROJECT_KEY=HVP-poc-crAPI-gateway
ENV SEEKER_AGENT_NAME=HVP-poc-agent-crAPI-gateway
RUN sh -c "$( curl -k -X GET -fsSL --header 'Accept: application/x-sh' 'https://poc271.seeker.synopsys.com/rest/api/latest/installers/agents/scripts/GO?osFamily=LINUX&downloadWith=curl&projectKey=HVP-poc-crAPI-gateway&webServer=GO_LINUX_DEFAULT&flavor=DEFAULT&agentName=&accessToken=')"
RUN chmod +x ./seeker/seeker-agent-linux-amd64
RUN ./seeker/seeker-agent-linux-amd64 go build -v -o server

RUN wget https://github.com/minio/certgen/releases/latest/download/certgen-linux-amd64 -O /tmp/certgen \
    && chmod +x /tmp/certgen

RUN /tmp/certgen -host "127.0.0.1,gateway-service,api.mypremiumdealership.com,mypremiumdealership.com"
RUN ls -la

FROM debian:buster-slim
WORKDIR /app

# set up env for seeker Go agent
ENV SEEKER_SERVER_URL=https://poc271.seeker.synopsys.com:443
ENV SEEKER_PROJECT_KEY=HVP-poc-crAPI-gateway
ENV SEEKER_AGENT_NAME=HVP-poc-agent-crAPI-gateway

RUN set -x && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Copy the binary to the production image from the builder stage.
COPY --from=builder /app/server /app/server
COPY --from=builder /app/public.crt /app/server.crt
COPY --from=builder /app/private.key /app/server.key

EXPOSE 443

CMD ["/app/server"]


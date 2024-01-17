# Licensed under the Apache License, Version 2.0 (the “License”);
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an “AS IS” BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


# GoLang Build
FROM golang:alpine AS builder
ENV GO111MODULE=on \
    CGO_ENABLED=0
WORKDIR /build
COPY ./go.mod .
COPY ./go.sum .
RUN go mod download
COPY ./main.go ./main.go
COPY ./api ./api
RUN go mod vendor
RUN ls -al

# build app with seeker agent
ENV SEEKER_SERVER_URL=https://poc271.seeker.synopsys.com:443
ENV SEEKER_PROJECT_KEY=HVP-poc-crAPI-community
ENV SEEKER_AGENT_NAME=HVP-poc-agent-crAPI-community
RUN apk add curl
RUN sh -c "$( curl -k -X GET -fsSL --header 'Accept: application/x-sh' 'https://poc271.seeker.synopsys.com/rest/api/latest/installers/agents/scripts/GO?osFamily=LINUX&downloadWith=curl&projectKey=HVP-poc-crAPI-community&webServer=GO_LINUX_DEFAULT&flavor=DEFAULT&agentName=&accessToken=')"
RUN chmod +x ./seeker/seeker-agent-linux-amd64
RUN ./seeker/seeker-agent-linux-amd64 go build -o main .
RUN go test -v ./...
WORKDIR /dist
RUN cp /build/main .

# Main Image
FROM alpine:3

# set env for seeker agent
ENV SEEKER_SERVER_URL=https://poc271.seeker.synopsys.com:443
ENV SEEKER_PROJECT_KEY=HVP-poc-crAPI-community
ENV SEEKER_AGENT_NAME=HVP-poc-agent-crAPI-community

# go
WORKDIR /app
RUN apk update && apk add --no-cache curl
COPY --from=builder /dist/main /app/main
COPY certs /app/certs
COPY health.sh /app/health.sh
RUN ls -al /app

ARG SERVER_PORT 
EXPOSE ${SERVER_PORT}
CMD /app/main

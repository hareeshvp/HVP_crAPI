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


# Java Maven Build
FROM gradle:7.3.3-jdk11 AS gradlebuild
WORKDIR /app
COPY *.gradle.kts ./
# Only download dependencies
# Eat the expected build failure since no source code has been copied yet
RUN gradle clean build --no-daemon > /dev/null 2>&1 || true
COPY src src
RUN gradle :spotlessApply
RUN gradle build
RUN gradle bootJar

# Main Image
FROM openjdk:11.0.15-jre-slim-buster

#Java
RUN apt-get -y update && apt-get -y install curl && apt-get -y clean
RUN mkdir /app
COPY --from=gradlebuild /app/build/libs/identity-service-1.0-SNAPSHOT.jar /app/identity-service-1.0-SNAPSHOT.jar

ARG SERVER_PORT 
EXPOSE ${SERVER_PORT}

ENV JAVA_TOOL_OPTIONS "-Xmx128m"

# add in seeker JAVA agent
ENV SEEKER_SERVER_URL=https://poc271.seeker.synopsys.com:443
ENV SEEKER_PROJECT_KEY=HVP-poc-crAPI-identity
ENV SEEKER_AGENT_NAME=HVP-poc-agent-crAPI-identity
COPY /openapi-spec/openapi-spec.json /openapi-spec/openapi-spec.json
RUN sh -c "$( curl -k -X GET -fsSL --header 'Accept: application/x-sh' 'https://poc271.seeker.synopsys.com/rest/api/latest/installers/agents/scripts/JAVA?osFamily=LINUX&downloadWith=curl&projectKey=HVP-poc-crAPI-identity&webServer=OTHER&flavor=DEFAULT&agentName=HVP-poc-agent-crAPI-identity&accessToken=')"
ENV JAVA_TOOL_OPTIONS="$JAVA_TOOL_OPTIONS -javaagent:$PWD/seeker/seeker-agent.jar"
ENV SEEKER_AGENT_APP_OPENAPI_SPEC_FILE="/openapi-spec/openapi-spec.json"

COPY jwks.json default_jwks.json
COPY entrypoint.sh /entrypoint.sh
COPY health.sh /app/health.sh
CMD [ "/entrypoint.sh"]

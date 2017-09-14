FROM node
EXPOSE 3000

RUN apt-get update && \
    apt-get -y install jq && \
    apt-get clean

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

RUN curl -L  https://github.com/IBM/watson-conversation-slots-intro/archive/master.tar.gz --output /tmp/master.tar.gz && \
    tar -xzvf /tmp/master.tar.gz -C /usr/src/app --strip-components 1

RUN npm install 

COPY *.sh /usr/src/app

CMD [ "/usr/src/app/run-server.sh"]

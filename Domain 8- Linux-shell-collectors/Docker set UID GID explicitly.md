#####  
https://github.com/docker-library/cassandra/blob/0e270a93b53119818f9c71fa273cda3a07d0bf5c/2.1/Dockerfile?spm=a2c65.11461447.0.0.47cd3b74HeBjnW

``````sh
# explicitly set user/group IDs
RUN groupadd -r cassandra --gid=999 && useradd -r -g cassandra --uid=999 cassandra

RUN groupadd -r sonarqube && useradd -r -g sonarqube sonarqube
``````



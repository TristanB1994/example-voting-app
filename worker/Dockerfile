FROM maven:3.6.1 AS compile-worker

WORKDIR /app

COPY . .

RUN mvn package -DskipTests \
    && mv target/worker-jar-with-dependencies.jar /run/worker.jar \
    && rm -rf *

FROM compile-worker as run-worker

CMD java -jar /run/worker.jar

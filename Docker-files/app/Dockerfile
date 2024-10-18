# build the artifact
FROM openjdk:11 AS build_image
LABEL "project"="vprofile"
LABEL "author"="Adenle"
RUN apt update && apt install maven -y
RUN git clone https://github.com/Adenleayo/vprofile-project.git
RUN cd vprofile-project && git switch docker && mvn install

# run the artifact from #BUILD_IMAGE
FROM tomcat:9-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalinah.sh","run"]
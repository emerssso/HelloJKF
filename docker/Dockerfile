FROM jenkins/jnlp-slave

USER root
RUN apt-get update -qq && apt-get install file -y
RUN chmod 777 /opt
RUN usermod -s /bin/sh jenkins
USER jenkins

#Install Android SDK
RUN cd /opt && wget -q https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip -O android-sdk.zip
RUN cd /opt && mkdir android-sdk-linux && unzip android-sdk.zip -d android-sdk-linux
RUN cd /opt && rm -f android-sdk.zip
ENV ANDROID_HOME /opt/android-sdk-linux
ENV PATH ${PATH}:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools:/opt/tools
RUN echo y | sdkmanager "tools" "platform-tools" | grep 'done'

WORKDIR /home/jenkins

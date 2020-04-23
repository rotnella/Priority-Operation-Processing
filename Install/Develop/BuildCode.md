# Building the Code

## MacOS

These instructions assume you are starting with nothing setup on your Mac.

1. Install brew (https://brew.sh/)
1. Install open jdk8
    ```    
    brew tap AdoptOpenJDK/openjdk
    brew cask install adoptopenjdk8
    ```
1. Install [Maven 3.3.9](https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.3.9/) (unzip)
    1. Setup your PATH to include (wherever you unzipped it)/apache-maven-3.3.9/bin
1. [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
1. Optional: [IDEA Community Edition](https://www.jetbrains.com/idea/download/#section=mac)
1. Build the code
   ```
   mvn -U clean install
   ```
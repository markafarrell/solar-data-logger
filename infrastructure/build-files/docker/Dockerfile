FROM ubuntu:latest

RUN apt update

RUN apt install -y wget

RUN apt install -y unzip

RUN wget https://releases.hashicorp.com/terraform/0.12.7/terraform_0.12.7_linux_amd64.zip

RUN unzip terraform_0.12.7_linux_amd64.zip

RUN mv terraform /usr/local/bin/

RUN terraform --version

# RUN apt install -y python3

# RUN apt install -y python3-pip

# RUN pip3 install virtualenv

WORKDIR /data
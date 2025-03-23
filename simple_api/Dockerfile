FROM python:3.8-buster

LABEL maintainer="Med Amine Moutaouakil <amine20991@gmail.com>"

COPY student_age.py /

RUN apt-get update -y && apt-get install -y python3-dev libsasl2-dev libldap2-dev libssl-dev

COPY requirements.txt /

RUN pip3 install -r requirements.txt

RUN mkdir /data

VOLUME /data

COPY student_age.json /data/student_age.json

EXPOSE 5000

CMD [ "python3", "./student_age.py" ]
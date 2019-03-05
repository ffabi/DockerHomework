#Use a pre-configured python image
FROM python:3.6-stretch
#Copy the requirements.txt file to the container
ADD . /code
WORKDIR /code
#Install needed python libraries
RUN pip3 install -r requirements.txt
#Set start up command
CMD ["python3", "app.py"]


FROM python:3.7

WORKDIR /usr/src/app

ENV PROJECT_HOME=/usr/src/app

COPY requirements.txt requirements.txt

RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install -r requirements.txt


COPY . .

CMD [ "python3", "predict_flask.py" ]

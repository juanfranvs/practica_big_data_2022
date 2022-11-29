# FBID2022 - Práctica sobre Predicción de Vuelos
Proyecto Realizado para la asignatura BDFI/FBID del MUIT/MUIRST de la ETSIT-UPM en el curso 2022/2023 por Juan Francisco Vara Sánchez y Jose Javier Mata de la Fuente.
Para el desarrollo de este proyecto tomamos como base el repositorio proporcionado: https://github.com/ging/practica_big_data_2019.

Realizaremos un despliegue en local partiendo y otro sobre Google Cloud utilizando la tecnología Docker para virtualizar los servicios en ficheros Dockerfile y lanzarlos con un fichero Docker Compose.

# Despliegue en Local
Primeramente descargamos el repositorio y accedemos a él:

    git clone https://github.com/ging/practica_big_data_2019.
    
Y nos movemos al repositorio:

    cd practica_big_data_2019

# Instalación de SW
Antes de realizar este despliegue nos centramos en instalar todas las herramientas software necesarias y que procedemos a detallar:
Por simplicidad, vamos a generar una carpeta software dentro del proyecto donde iremos colocando de forma ordenada el software necesario

- **Intellij**
Visitamos la página de JetBrains y nos descargamos el código asociado a la versión Linux que necesitamos https://www.jetbrains.com/idea/download/#section=linux
- **Python3**
 
        sudo apt install python 3.7    
- **PIP**

        sudo apt-get install python3-pip
- **SBT**

        sudo apt install sbt
- **MongoDB**

        wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
        sudo apt-get install gnupg
        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
        sudo apt-get update
        sudo apt-get install -y mongodb-org
        echo "mongodb-org hold" | sudo dpkg --set-selections
        echo "mongodb-org-database hold" | sudo dpkg --set-selections
        echo "mongodb-org-server hold" | sudo dpkg --set-selections
        echo "mongodb-mongosh hold" | sudo dpkg --set-selections
        echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
        echo "mongodb-org-tools hold" | sudo dpkg --set-selections
Para arrancar el servicio de mongo y acceder a la shell de Mongo ejecutamos:
        sudo systemctl start mongod
        mongosh
- **Spark y Scala**
Descargamos el paquete https://mirrors.estointernet.in/apache/spark/spark-3.1.2/ (Versión 3.1.2 de Spark que incluye Hadoop 3.2)

        cd /practica_big_data_2019/software
        spark-3.1.2-bin-hadoop3.2.tgz
        tar -xzf spark-3.1.2-bin-hadoop3.2.tgz   
- **Kafka**
  Instlación de la  versón kafka_2.12-3.0.0 a través de la URL: https://kafka.apache.org/quickstart
  
        cd /practica_big_data_2019/software 
        kafka_2.12-3.0.0.tgz
        tar -xzf kafka_2.12-3.0.0.tgz
        
## Downloading Data
Nos movemos al **directorio de la práctica**

        cd practica_big_data_2019
Descargamos los datos ejecutando:

        ./resources/download_data.sh
## Install Python libraries

        pip install -r requirements.txt
## Start Zookeeper
Arrancamos en un terminal el servidor de Zookeeper:

        bin/zookeeper-server-start.sh config/zookeeper.properties
## Start Kafka
Arrancamos en un terminal el servidor de Kafka:

        bin/kafka-server-start.sh config/server.properties
Creamos en un nuevo terminal un **Tópico**

        bin/kafka-topics.sh \
            --create \
            --bootstrap-server localhost:9092 \
            --replication-factor 1 \
            --partitions 1 \
            --topic flight_delay_classification_request
Nos devuelve el siguiente mensaje:

         Created topic "flight_delay_classification_request".
Una vez creado mostramos la **Lista de Tópicos**:

         flight_delay_classification_request
Opcionalmente podemos generar un **Consumidor**:

         bin/kafka-console-consumer.sh \
            --bootstrap-server localhost:9092 \
            --topic flight_delay_classification_request \
            --from-beginning
## Import the distance records to MongoDB
Comprobamos si la máquina está activa ejecutando:

         sudo service mongod status
Nos devuelve:

     mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor pr>
     Active: active (running) since Tue 2022-11-15 16:53:03 CET; 23h ago
     Docs: https://docs.mongodb.org/manual
     Main PID: 18011 (mongod)
     Memory: 50.4M
     CGroup: /system.slice/mongod.service
            └─18011 /usr/bin/mongod --config /etc/mongod.conf
            
Ejecutamos el script import_distances.sh

          ./resources/import_distances.sh
Nos devuelve:

     2019-10-01T17:06:46.957+0200	connected to: mongodb://localhost/
     2019-10-01T17:06:47.035+0200	4696 document(s) imported successfully. 0 document(s) failed to import.
     MongoDB shell version v4.2.0
     connecting to: mongodb://127.0.0.1:27017/agile_data_science?compressors=disabled&gssapiServiceName=mongodb
     Implicit session: session { "id" : UUID("9bda4bb6-5727-4e91-8855-71db2b818232") }
     MongoDB server version: 4.2.0
        {
	        "createdCollectionAutomatically" : false,
	        "numIndexesBefore" : 1,
	        "numIndexesAfter" : 2,
	        "ok" : 1
        }

   
            
## Train and Save the model with Pyspark mllib
Dentro del directorio de la práctica exportamos las variables de entorno:

          export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
          export SPARK_HOME=/opt/spark
Entrenamos el modelo dentro de practica_big_data_2019/resources:

          python3 resources/train_spark_mllib_model.py .
Vemos el modelo generado moviendonos al directorio models
        
          ls ../models
## Run Flight Predictor
Añadimos la siguiente sentencia dentro del fichero MakePrediction.scala:

          cd /flight_prediction/src/main/scala/es/upm/dit/ging/predictor
          val base_path="/home/upm/practica_big_data_2019"
Una vez realizado el cambio abrimos Intellij compilamos y ejecutamos el fichero MakePrediction

	  cd /opt/idea-IC-222.4345.14/bin 
	  ./idea.sh

## Start the prediction request Web Application
Añadimos la variable de entorno PROJECT_HOME que contendrá la ruta de la práctica:

          export PROJECT_HOME="/home/upm/practica_big_data_2019
Vamos a la ruta web del proyecto y ejecutamos el fichero predict_flask.py:

          cd practica_big_data_2019/resources/web
          python3 predict_flask.py
Este fichero nos permitirá abrir la web predictora de vuelos que arranca la aplicación sobre el puerto 5000 de localhost: http://localhost:5000/flights/delays/predict_kafka y ejecutar el botón Submit

## Check the predictions records inserted in MongoDB
Una vez realizamos la predicción con el interfaz web abrimos la shell de MongoDB para ver que los datos realizados por la predicción son almacenados de forma correcta.

           > mongosh
           > use agile_data_science
           > db.flight_delay_classification_response.find()
Esto nos devuelve como respuesta una query de Mongo que contiene los datos que se han almacenado de dicha predicción.

             {
                _id: ObjectId("6373cc430e39684b17440328"),
                Origin: 'SFO',
                DayOfWeek: 1,
                DayOfYear: 359,
                DayOfMonth: 25,
                Dest: 'ATL',
                DepDelay: 5,
                Timestamp: ISODate("2022-11-15T17:28:35.113Z"),
                FlightDate: ISODate("2018-12-24T23:00:00.000Z"),
                Carrier: 'AA',
                UUID: 'efab3289-49ce-4233-9c47-4983fb3acc91',
                Distance: 2139,
                Route: 'SFO-ATL',
                Prediction: 2
             }

## Ejecución con Spark-Submit
Una vez que hemos probado con Intellij la predicción, paramos el código de MakePrediction y procedemos a generar un fichero con extensión .jar que se encargará de compilar y ejecutar el código empleando el comando **spark-submit**.
Vamos a la ruta /flight_prediction/target/scala y ejecutamos:

            sbt clean
            sbt package
Con este fichero ejecutamos el siguiente comando:

            spark-submit --class es.upm.dit.ging.predictor.MakePrediction --master local --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1 /home/upm/practica_big_data_2019/flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar
Con ello, podremos volver a generar otra predicción que también se almacenará en la BBDD de MongoDB sin necesidad de utilizar Intellij.

## Apache Airflow(Opcional)
Con este SW de Apache podremos entrenar el modelo de otra manera. 
The version of Apache Airflow used is the 2.1.4 and it is installed with pip. For development it uses SQLite as database but it is not recommended for production. For the laboratory SQLite is sufficient.
Install python libraries for Apache Airflow (suggested Python 3.7)

	cd resources/airflow
	python3.7 -m pip install -r requirements.txt -c constraints.txt
Seleccionamos la variable de entorno PROJECT_HOME cogiendo la ruta del repositorio de la práctica
	
	export PROJECT_HOME=/home/upm/practica_big_data_2019
Establecemos un nuevo valor de la variable PATH:
	
	export PATH=$PATH:~/.local/bin
En un terminal, configuramos el environment de airflow:

	export AIRFLOW_HOME=~/airflow
	mkdir $AIRFLOW_HOME/dags
	mkdir $AIRFLOW_HOME/logs
	mkdir $AIRFLOW_HOME/plugins

	airflow users create \
    	--username admin \
    	--password admin \
    	--firstname Jack \
    	--lastname  Sparrow\
    	--role Admin \
    	--email example@mail.org
Iniciamos la base de datos:

	airflow db init
Iniciamos en otro terminal el scheduler y el webserver de airflow:

	airflow webserver --port 8080
	airflow scheduler
	
Añadimos y ejecutamos el DAG alojado en 				**resources/airflow/setup.py**

Nos conectamos a la página  http://localhost:8080/home y nos muestra la lista de DAGs accesibles
Si pinchamos sobre la solapa de **Play->Trigger Dag** nos entrena un modelo que aparece representado en la lista de DAGs **Runs**. Pinchamos sobre el DAG y no muestra el histórico de predicciones que hemos realizado.

**TODO**: add the DAG and execute it to train the model (see the official documentation of Apache Airflow to learn how to exectue and add a DAG with the airflow command). Para poder añadir el DAG se ha necesitado ejecutar el siguiente comando desde la carpeta practica_big_data_2019:

Para añadir el dag tenemos que copiar el fichero alojado en la ruta **resources/airflow/setup.py** y extraerlo fuera de la carpeta del repositorio. Para ello ejecutamos el comando:

	sudo cp /resources/airflow/setup.py /home/upm/airflow/dags
Una vez copiado el fichero que contiene el DAG nos autenticamos por el interfaz web a Apache Airflow introduciendo las credenciales user:admin y contraseña:admin accedemos a la lista de DAGs existentes en la web.

**TODO**: explain the architecture of apache airflow (see the official documentation of Apache Airflow)

Airflow trabaja con DAGs(Directed Aciclic Graph) como una metodología para estructurar los procesos por lotes que se van a ejecutar en un flujo de trabajo mediante relaciones y dependencias. Estos grafos deberán cumplir 2 condiciones:

 - Acíclicos: No deben existir bucles por lo que la ejecución de un nodo no puede regresar a otro nodo que ya ha sido ejecutado.
 - Dirigidos: Las relaciones de los nodos son de un único sentido.
 
Dentro de la estructura que presenta Apache Airflow destacamos 

 - Scheduler: Encargado de los flujos de trabajo planificados
 - Executor: Pone en contacto a los componentes de la arquitectura con el worker que asigna tareas.
 - DAG Directory: Directorio que almacena las tareas descritas por ficheros Python.
 - Workers: Donde se almacenan las tareas.
 - Metadata Database: Base de Datos que contiene metadatos.
 - Webserver: Servidor que presenta la interfaz.

**TODO**:  analyzing the setup.py: what happens if the task fails?, what is the peridocity of the task?

Observando el fichero **setup.py** dentro del array default_args en su propiedad retries y retry_delay vemos que presentan los valores 3 y timedelta(minutes=5) que indica que el DAG se intenta iniciar hasta 3 veces teniendo una **periodicidad de task de 5 minutos entre cada intento**. Además viendo el código vemos que no está configurado para que se inicie la tarea periódicamente.

# DOCKER
A partir de ahora vamos a generar contenedores para poder desplegar todos los servicios sobre Google Cloud. Para ello realizamos el proceso siguiente:

## Creamos la red

	sudo docker network create mynet --driver bridge
	sudo docker run -d  --network mynet --name mongopr_bd -v /home/josejaviermata7/practica_big_data_2019:/home/practica_big_data_2019 -p 27017:27017 mongo
## Ejecutamos el fichero import_distances.sh

	docker exec -w /home/practica_big_data_2019 mongopr_bd ./resources/import_distances.sh
	sudo docker exec -it mongopr_bd mongosh agile_data_science
	show collections
## Desplegamos Zookeeper

	sudo docker run -d --name zookeeper \
 		--network mynet\
 		-p 2181:2181 \
 		-e ALLOW_ANONYMOUS_LOGIN=yes \
 		 bitnami/zookeeper:latest
## Desplegamos Kafka

	sudo docker run -d --name kafka \
 	--network mynet\
 	-p 9092:9092 \
 	-e KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 \
	-e ALLOW_PLAINTEXT_LISTENER=yes \
 	-e KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT \
 	-e KAFKA_CFG_LISTENERS=CLIENT://:9092 \
 	-e KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092 \
 	-e KAFKA_CFG_INTER_BROKER_LISTENER_NAME=CLIENT \
 	bitnami/kafka:3.0.0
## Generamos el topic

	sudo docker exec kafka kafka-topics.sh \
 	--create \
 	--bootstrap-server kafka:9092 \
 	--replication-factor 1 \
 	--partitions 1 \
 	--topic flight_delay_classification_request
### Comprobamos la generación del topic

	sudo docker exec kafka kafka-topics.sh --bootstrap-server kafka:9092 --list
## Arrancamos el spark-master

	sudo docker run -d --name spark-master \
  	--network=mynet \
  	-e SPARK_MODE=master \
  	-p 10.204.0.2:8080:8080 \
	bitnami/spark:3.1.2
	
## Arrancamos el spark-worker

	sudo docker run -d --name spark-worker \
  	--network=mynet \
  	-e SPARK_MODE=worker \
  	-p 10.204.0.2:8081:8081 \
	-v /home/josejaviermata7/practica_big_data_2019:/home/practica_big_data_2019 \
	-v /home/josejaviermata7/jars:/opt/bitnami/spark/.ivy2:z \
	bitnami/spark:3.1.2
	
### Ejecutamos Spark-submit 
	sudo docker exec spark-worker spark-submit --master spark://5725f561d1dd:7077 --deploy-mode cluster --packages org.mongodb.spark:mongo-spark-			connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.0 --class es.upm.dit.ging.predictor.MakePrediction 		
	/home/practica_big_data_2019/flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar
	
### Dockerfile para la aplicación flask
Generamos el Dockerfile que se puede ver a continuación y el cual hemos usado para el posterior despliegue de la aplicación web tras la creación del contenedor mediante el comando docker build.

FROM python:3.7

WORKDIR /usr/src/app

ENV PROJECT_HOME=/usr/src/app

COPY requirements.txt requirements.txt

RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install -r requirements.txt


COPY . .

CMD [ "python3", "predict_flask.py" ]
	
### Lanzamos el Predict Flask

	sudo docker run -p 10.204.0.2:5000:5000 flask








	
	
	

            
        
        

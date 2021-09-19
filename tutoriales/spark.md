## ¿Que es Spark?

Spark es un framework de trabajo para el desarrollo de grandes datos o big data y se preocupa de la velocidad y continuidad del procesamiento de datos, en contraparte de Hadoop que se preocupa por un almacenamiento grande de datos.

Podemos utilizar multiples lenguajes

- Java
- Scala (Spark corre nativamente aquí)
- Python
- R

¿Que no es Apache Spark?

No es una base de datos

**OLAP:** Base de datos tradicional en tiempo real

Spark debe estar conectado a un Data warehouse para poder aprovechar toda su funcionalidad

#### Historia

- Nace en 2009 en la Universidad Berkeley
- Hereda de Hadoop
- Version3 fue liberada en Junio 2020

#### Spark VS Hadoop

- Spark se enfoca en procesamiento de datos desde la memoria ram.

- Posee naturalmente un modulo para ML, streaming y grafos.

- No depende de un sistema de archivos.

## Configuración

### Instalación del ambiente de trabajo

Instalaremos lo siguiente en linux/wsl2

- Agregamos java8 usando open jdk

```bash
sudo ad-apt-repository ppa:openjdk-r/ppa
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get -y install openjdk-8-jre
```

- Python 3.7 esto porque spark 2.4 no tiene soporte aun para python 3.8

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.7
```

- Scala

```bash
sudo apt-get -y install scala
```

- Pip3

```bash
sudo apt-get -y install python3-pip
sudo pip3 install py4j
#traduce condigo python a java
```

#### Instalacion de  Apache Spark

En la pagina de Apache Spark <https://spark.apache.org/downloads.html>

Seleccionamos la opción Spark release 2.4.1, con el Pre-built for Apache Hadoop 2.7, y después haces click en Download.

Descarga directo a wsl con curl o de forma habitual con windows

```bash
curl -o spark-2.4.7-bin-hadoop2.7.tgz https://downloads.apache.org/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz
```

Descomprimir el archivo (debes estar en el folder donde se encuentre descargado tu archivo)

```bash
tar -xvf spark-2.4.7-bin-hadoop2.7.tgz

#verificamos
ls
```

Renombramos la carpeta/directorio

```bash
mv spark-2.4.7-bin-hadoop2.7 spark
```

movemos la carpeta a home

```bash
mv spark ~/
```

Borramos el archivo tgz que descargamos

```bash
rm spark-2.4.7-bin-hadoop2.7.tgz
```

#### Instalacion de Anaconda

```bash
# Descargamos
wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh

# Instalamos
bash Anaconda3-2020.02-Linux-x86_64.sh

# Do you wish the installer to initialize Anaconda3
# by running conda init? [yes|no]
# [no] >>> no
```

#### Instalamos py4j en Anaconda

Para poder hacer uso del instalador de anaconda exportamos la siguiente variable

```bash
# la variable $USER para acceder tu nombre de usuario
export PATH=/home/$USER/anaconda3/bin:$PATH

# Instalamos py4j
conda install py4j
```

### Clase 5 Jupyter vs CLI: ejecucion de Spark desde la linea de comandos

Hecha la instalacion de Spark agregaremos varias variables de entorno en el archivo .bashrc, vamos al final del archivo y agregamos lo siguiente

**Nota:** Si eres estudiante de platzi, y por alguna razón usas zshell  y no la bash (la terminal por defecto sin customizar) deberás incluir la configuracion en el archivo .zshrc y no en el archivo .bashrc.

```bash
sudo nano .bashrc

## Path de Java
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
export PATH=$JAVA_HOME:$PATH

## Spark
export SPARK_HOME='/home/'$USER'/spark'
export PATH=$SPARK_HOME:$PATH

## Python para ser utilizable por Spark
export PYTHONPATH=$SPARK_HOME/python:$PYTHONPATH
export PYSPARK_PYTHON=python3.7

# Guardamos el archivo con ctrl + o
# recargamos el archivo
source .bashrc
```

utilizé la variable de entorno $USER para que no se tenga que ajustar a cada instalacion para aquellas personas que aun no utilizan linux de forma habitual y esto solo agrega el nombre de usuario que tienes en la terminal al path.

Hecho lo anterior vemos los binarios que Spark posee.

```bash
ls ~/spark/bin/
```

En los binarios observamos los siguientes.

- **pyspark**  permite ejecutar código en vivo  como un interprete
- **spark-submit** permite ejecutar un script como cualquier archivo.py

Descargamos el repositorio <https://github.com/terranigmark/curso-apache-spark-platzi>

Utilizamos los archivos data.csv y codeExample.py

```bash
head -n 10 data.csv

# output
State,Color,Count
TX,Red,20
NV,Blue,66
CO,Blue,79
OR,Blue,71
WA,Yellow,93
WY,Blue,16
CA,Yellow,53
WA,Green,60
OR,Green,71
```

```py
import sys

from pyspark.sql import SparkSession
from pyspark.sql.functions import count

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: mnmcount <file>", file=sys.stderr)
        sys.exit(-1)

    spark = (SparkSession
        .builder
        .appName("PythonMnMCount")
        .getOrCreate())
    # get the M&M data set file name
    mnm_file = sys.argv[1]
    # read the file into a Spark DataFrame
    mnm_df = (spark.read.format("csv")
        .option("header", "true")
        .option("inferSchema", "true")
        .load(mnm_file))
    mnm_df.show(n=5, truncate=False)
    # aggregate count of all colors and groupBy state and color
    # orderBy descending order
    count_mnm_df = ( mnm_df.select("State", "Color", "Count")
                    .groupBy("State", "Color")
                    .agg(count("Count")
                    .alias("Total"))
                    .orderBy("Total", ascending=False))
    # show all the resulting aggregation for all the dates and colors
    count_mnm_df.show(n=60, truncate=False)
    print("Total Rows = %d" % (count_mnm_df.count()))
    #
    # find the aggregate count for California by filtering
    ca_count_mnm_df = ( mnm_df.select("*")
                       .where(mnm_df.State == 'CA')
                       .groupBy("State", "Color")
                       .agg(count("Count")
                            .alias("Total"))
                       .orderBy("Total", ascending=False) )

    # show the resulting aggregation for California
    ca_count_mnm_df.show(n=10, truncate=False)

```

Instala la dependencia **pyspark**

```bash
conda install pyspark
```

Ejecutamos el código con spark-submit y optemos los resultados

```bash
~/spark/bin/spark-submit codeExample.py data.csv
```

Solucionar el primer warning descarga los binarios de hadoop (version 2.7.3, puedes probar 2.10.0)

<https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz>

Y ejecuta lo siguiente en la terminal

```bash
# descomprimir
tar -xvf hadoop-2.7.3.tar.gz

#Renombramos la carpeta/directorio
mv hadoop-2.7.3.tar.gz hadoop

# movemos la carpeta a home
mv hadoop ~/
```

Agrega las siguientes variables de entorno con en la clase anterior, lo agregue después de las de java y antes de Spark

```bash
export HADOOP_HOME='/home/'$USER'/hadoop'
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME'/lib/native/libhadoop.so.1.0.0'
```
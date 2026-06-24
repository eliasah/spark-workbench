FROM eclipse-temurin:11-jdk-focal

ENV SPARK_VERSION=3.5.1 \
    SCALA_VERSION=2.12.18 \
    SPARK_HOME=/opt/spark \
    SCALA_HOME=/opt/scala \
    PYTHONPATH=/opt/spark/python:/opt/spark/python/lib/py4j-0.10.9.7-src.zip \
    PYSPARK_PYTHON=python3 \
    PATH="/opt/spark/bin:/opt/scala/bin:/usr/local/bin:${PATH}"

RUN apt-get update && apt-get install -y --no-install-recommends \
        python3.8 \
        python3.8-dev \
        python3-pip \
        procps \
    && rm -rf /var/lib/apt/lists/* \
    && update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1 \
    && update-alternatives --install /usr/bin/python  python  /usr/bin/python3.8 1

# Tarballs are pre-downloaded into downloads/ before the build (see README).
# Using COPY avoids network calls inside Docker and makes the build reliable.
COPY downloads/spark-${SPARK_VERSION}-bin-hadoop3.tgz /tmp/spark.tgz
RUN tar -xz -C /opt -f /tmp/spark.tgz \
    && mv "/opt/spark-${SPARK_VERSION}-bin-hadoop3" "${SPARK_HOME}" \
    && rm /tmp/spark.tgz

COPY downloads/scala-${SCALA_VERSION}.tgz /tmp/scala.tgz
RUN tar -xz -C /opt -f /tmp/scala.tgz \
    && mv "/opt/scala-${SCALA_VERSION}" "${SCALA_HOME}" \
    && rm /tmp/scala.tgz

COPY wheels/ /wheels/

RUN pip3 install --no-index --find-links=/wheels \
        pandas==2.0.3 \
        pyarrow==17.0.0 \
        pyspark==3.5.1 \
        notebook

WORKDIR /workspace

EXPOSE 8888 4040

CMD ["jupyter", "notebook", \
     "--no-browser", \
     "--ip=0.0.0.0", \
     "--port=8888", \
     "--allow-root", \
     "--NotebookApp.token=", \
     "--notebook-dir=/workspace/notebooks"]

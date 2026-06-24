FROM apache/spark:3.5.1-scala2.12-java11-python3-ubuntu

USER root

# Install curl (needed for standalone Scala below) and clean apt cache
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

# Jupyter + data science packages (pyspark already on PYTHONPATH via SPARK_HOME)
RUN pip install --no-cache-dir \
    pandas==2.0.3 \
    pyarrow==17.0.0 \
    "jupyterlab==4.2.5" \
    "notebook==7.2.2"

# Standalone scala REPL — spark-shell is already bundled; this adds `scala`
ENV SCALA_HOME=/opt/scala \
    PATH="/opt/scala/bin:${PATH}"

RUN curl -fsSL -o /tmp/scala.tgz \
      https://scala-lang.org/files/archive/scala-2.12.18.tgz \
    && tar -xz -C /opt -f /tmp/scala.tgz \
    && mv /opt/scala-2.12.18 /opt/scala \
    && rm /tmp/scala.tgz

WORKDIR /workspace
EXPOSE 8888 4040

CMD ["jupyter", "notebook", \
     "--no-browser", \
     "--ip=0.0.0.0", \
     "--port=8888", \
     "--allow-root", \
     "--NotebookApp.token=", \
     "--notebook-dir=/workspace/notebooks"]

# my_docker_jupyter_notebook
Run Jupyter Notebook inside a Docker container

## Step 1: Clone this repository
```bash
git clone https://github.com/hillwu7/jupyter_notebook_docker.git
cd jupyter_notebook_docker
```

## Step 2: Set up the docker-compose file
The `docker-compose.yaml` file is already created. Ensure it contains the following content:

```yaml
version: '3'

services:
  jupyter_notebook_docker:
    build:
      context: ./jupyter_notebook_server
      dockerfile: Dockerfile
    image: jupyter_notebook_docker:1.0
    container_name: jupyter_notebook_docker
    ports:
      - 8888:8888
    volumes:
      - ./jupyter_notebook_server/sync_note:/app/sync_note

```

## Step 3: Start the Docker container with docker-compose
Make sure to change the path to the root of the `docker-compose.yaml` folder and run the following command:

```bash
docker-compose up 
```

This command builds the Docker image (if not already built) and starts the Jupyter Notebook server. Access the notebook at [http://localhost:8888/tree?token=jupyter_notebook_docker_token](http://localhost:8888/tree?token=jupyter_notebook_docker_token) in your web browser.

## Step 5: Stop and remove all services (Optional)
To stop and remove all services, use the following command:

```bash
docker-compose down 
```

This will shut down the running containers and remove associated networks and volumes.

## Alternative: Run with docker run command
If you prefer to run the container using docker run directly, you can use the following command,make sure you have built the docker image first:

```bash
docker run -p 8888:8888 -v ./jupyter_notebook_server/sync_note:/app/sync_note -e JUPYTER_TOKEN=jupyter_notebook_docker_token jupyter_notebook_docker:1.0
```

This command achieves the same result as the `docker-compose` setup. It maps the local `./jupyter_notebook_server/sync_note` directory to `/app/sync_note` inside the container and sets the Jupyter Notebook token.

### Additional Information:
- Change the name of the image fetched from the repository with the `docker-compose.yaml` file at `image: jupyter_notebook_docker:`.
- The Jupyter Notebook token is set to `jupyter_notebook_docker_token`. Modify it by changing the `token` variable in the `config` folder `setting.yaml` file.
- Notebooks are synced to the `/sync_note` directory inside the Docker container and mapped to `./jupyter_notebook_server/sync_note` on your host machine. Adjust the host data path according to your preference.

---

## Docker Image Information
The Docker image is built based on `FROM` part at `DockerFile` as the parent image, yor can change it ify ou need.

The `environment.yaml` file under the `config` folder is used to create a Conda environment, which is activated during the image build.

The `Jupyter Notebook` is configured to run on port `8888`, Modify it by changing the `token` variable in the `config` folder `setting.yaml` file.

### Dockerfile
```Dockerfile
# Use a specific version of Miniconda3 as a parent image
FROM continuumio/miniconda3:23.10.0-1

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY config /app/config

# Create a Conda environment from environment.yaml and activate it
RUN conda env create -f /app/config/environment.yaml && \
    conda run -n $(grep name /app/config/environment.yaml | awk '{print $2}') \
    /bin/bash -c "python -m ipykernel install --user --name=$(grep name /app/config/environment.yaml | awk '{print $2}') --display-name=$(grep name /app/config/environment.yaml | awk '{print $2}')" && \
    echo "export PATH=/opt/conda/envs/$(grep name /app/config/environment.yaml | awk '{print $2}')/bin:$PATH" >> /etc/profile.d/conda.sh && \
    rm -rf /var/lib/apt/lists/*

# Run Jupyter Notebook
CMD ["/bin/bash", "-c", "source activate $(grep name /app/config/setting.yaml | awk '{print $2}') \
        && jupyter notebook --ip=0.0.0.0 --port=$(grep port /app/config/setting.yaml | awk '{print $2}') \
        --no-browser --allow-root --NotebookApp.token=$(grep token /app/config/setting.yaml | awk '{print $2}')"]

```

---

### Image contents:
---

`jupyter_notebook_docker:1.0`

```yaml
name: jupyter_notebook_docker
channels:
  - defaults
dependencies:
  - jupyter=1.0.0
  - pandas=2.1.4
  - plotly=5.9.0
  - pyspark=3.4.1
  - python=3.10.13
```

```yaml
token: jupyter_notebook_docker_token
port: 8888
```

---


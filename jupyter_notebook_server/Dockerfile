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

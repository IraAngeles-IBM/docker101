# Lab 2- Adding Value with Custom Docker Images

### Login to Docker Hub

1. Navigate to https://hub.docker.com and create an account if you haven't already

    For this lab we will be using the docker hub as our central registry. Docker hub is a free service to store publicly available images, or you can pay to store private images. Go to the [DockerHub](https://hub.docker.com) website and create a free account.

    Most organizations that use docker heavily will set up their own registry internally. To simplify things, we will be using the Docker Hub, but the following concepts apply to any registry.

1. Save your DockerHub username into a variable, so that you can copy/paste the rest of the commands for this lab. Replace the command below with your username.

    ```sh
    $ export DOCKER_USER=[docker username]
    ```

1. Login

    You can log into the docker registry account by typing `docker login` on your terminal.

    ```sh
    $ docker login -u $DOCKER_USER
    Password:
    ```

## Step 1: Create a python app (without using Docker)

1. Create a folder for this project, If you are using a cloud shell use:

    ```sh
    $ mkdir myapp
    $ cd myapp
    ```

1. Run the following command to create a file named `app.py` with a simple python program. (copy-paste the entire code block)

    ```bash
    echo 'from flask import Flask

    app = Flask(__name__)

    @app.route("/")

    def hello():
    return "hello world!"

    if __name__ == "__main__":
        app.run(host="0.0.0.0")' > app.py
    ```

    This is a simple python app that uses flask to expose a http web server on port 5000 (5000 is the default port for flask). Don't worry if you are not too familiar with python or flask, these concepts can be applied to an application written in any language.


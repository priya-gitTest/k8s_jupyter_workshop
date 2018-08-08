# Step 3 - Building your first docker image

In the [last lesson](part02.md) you found that the dependencies of the workshop-in-a-workshop were an installed python 3, pyqrcode, pymaging and pymaging-png.

These were installed using:

* pyqrcode: `pip install qrcode[pil]`
* pymaging: `pip install git+git://github.com/ojii/pymaging.git#egg=pymaging`
* pymaging-png: `pip install git+git://github.com/ojii/pymaging-png.git#pymaging-png`

For someone to run this workshop they would need to;

1. Download and install anaconda python
1. Install the above dependencies using pip
1. Obtain a copy of all of the notebooks in the example_workshop directory

## Building containers using Docker

Containers provide a great way to package together software and files into a single package that can be easily installed and run by end users. [Docker](https://docker.com) is the most prominant example of a containerisation platform.

With docker, you use a `DockerFile` to specify what software must be installed in the container, and what files are needed. The docker application then packages all of these files and software into a single container that can be easily downloaded and run by end users.

This workshop will not teach Docker. You can learn more about docker by following [one](https://docs.docker.com/get-started/) [of](https://docker-curriculum.com/) [these](https://www.tutorialspoint.com/docker/index.htm) [tutorials](https://stackify.com/docker-tutorial/).

Below is a fully commented `DockerFile` that can be used to build a container for the workshop-in-a-workshop.

```
# Start your container from one of the many example 
# containers on DockerHub. In this case, we will start
# with a minimal container that provides a working
# Python 3.6 environment

FROM python:3.6-slim-stretch

# Next, it is good to label your image with the maintainer
# Please feel free to put your own name and contact 
# address below
LABEL maintainer="Christopher Woods <Christopher.Woods@bristol.ac.uk>"

# First we want to install all of the Python dependencies
# We should do this as root
USER root

# We will need git to install the Pymaging dependencies.
# It is good practice to update apt before the install,
# and to then remove any unnecessary files afterwards so
# that you keep your container size small
RUN apt-get update && \
    apt-get install --no-install-recommends -qy git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Now lets run the three pip commands
RUN pip install qrcode[pil]
RUN pip install git+git://github.com/ojii/pymaging.git#egg=pymaging
RUN pip install git+git://github.com/ojii/pymaging-png.git#pymaging-png

# We also need to ensure we install jupyter (it is required to
# run a workshop as a jupyter notebook!)
RUN pip install jupyter

# We don't want to be root when running the container,
# so lets now create a normal user account called "jovyan"
# To do this, first create some environment variables
# for this user
ENV SHELL=/bin/bash \
    NB_USER=jovyan \
    NB_UID=1000 \
    NB_GID=100

ENV HOME=/home/$NB_USER

# Now add a little script that can properly set the file
# access permissions of directories - put it into /usr/bin
ADD fix-permissions /usr/bin/fix-permissions

# We are now ready to create the user account...
RUN useradd -m -s $SHELL -N -u $NB_UID $NB_USER && \
    fix-permissions $HOME

# Now change into the HOME directory and switch
# into the $NB_USER account. All commands from now
# will execute as the user
WORKDIR $HOME
USER $NB_USER

# Add all of the workshop files to the home directory
ADD example_workshop/lesson*.ipynb $HOME/

# The jupyter notebook will run on port 8888. Make sure that
# this port is exposed by the container
EXPOSE 8888

# Set the entrypoint of the container. This is the command
# that will be run when the container starts.
ENTRYPOINT ["jupyter-notebook", "--ip=0.0.0.0"]

# Always finish a Dockerfile by changing to a normal user.
# This stops you from ever publishing a container that accidentally
# runs as root!
USER $NB_USER
```

```
docker build . -t workshop
docker run --rm -p 8888:8888 workshop
```


***

# [Previous](part02.md) [Up](../README.md) [Next](part04.md)

FROM python:3.10-bookworm

# install system dependencies required by Whisper-AI
RUN apt update \
 && apt install --yes vim-tiny less ffmpeg \
 && apt --yes remove python3.11 python3.11-minimal \
 && apt --yes autoremove

ARG USER_NAME=vscode
ARG USER_UID=1000
ARG USER_GID=${USER_UID}

# Create the user
RUN groupadd --gid $USER_GID ${USER_NAME} \
 && useradd --uid ${USER_UID} --gid ${USER_GID} --shell /usr/bin/bash --create-home ${USER_NAME}

USER ${USER_UID}:${USER_GID}

# This runs cool if you open a shell
# - pros: the venv is in a user writable folder
# - cons: I don't know how to instruct vscode to always activate the venv
# RUN cd /home/vscode \
#  && python3.10 -m venv whisper-webui.venv \
#  && . whisper-webui.venv/bin/activate \
#  && pip install --upgrade pip \
#  && pip install -r requirements.txt
# RUN echo 'source /home/vscode/whisper-webui.venv/bin/activate' >> /home/vscode/.bashrc

# This is a replacement to the above method in environments where requirements.txt is stable enough
# - pros: requiremens are available to the entire container not matter whether they run bash or not
# - cons: you need to have a requirements.txt file ready and users won't be able to install further dependencies
COPY requirements.txt /home/vscode/requirements.txt
RUN cd /home/vscode \
 && pip install --upgrade pip \
 && pip install -r requirements.txt

WORKDIR /home/vscode

ARG REMOTEREF=refs/heads/main

RUN curl -L -o /tmp/whisper-webui.zip https://github.com/tchinchow/whisper-webui/archive/${REMOTEREF}.zip \
 && unzip /tmp/whisper-webui.zip -d /home/vscode \
 && rm /tmp/whisper-webui.zip \
 && mv /home/vscode/whisper-webui-* /home/vscode/whisper-webui \
 && diff /home/vscode/requirements.txt /home/vscode/whisper-webui/requirements.txt \
 && cd /home/vscode/whisper-webui \
 && cp .env.example .env \
 && echo -e "\nVERSION=${REMOTEREF}" >> .env

WORKDIR /home/vscode/whisper-webui

ENV GRADIO_SERVER_NAME=0.0.0.0
ENV GRADIO_SERVER_PORT=7860
EXPOSE 7860
ENTRYPOINT ["/usr/local/bin/python", "app.py"]
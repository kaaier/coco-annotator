FROM node:10 as build-stage

WORKDIR /workspace/
COPY ./client /workspace/client

RUN npm install -g --quiet \
        @vue/cli@3.3.0 \
        @vue/cli-service@3.3.0

COPY ./client/package* /workspace/

RUN npm install
ENV NODE_PATH=/workspace/node_modules

WORKDIR /workspace/client
RUN npm run build

# Setup flask

FROM python:3.6 as production-stage

RUN apt-get update && \
        apt-get install -y \
        wget \
        nano

WORKDIR /workspace

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt && \
        pip install gunicorn[eventlet]==19.9.0 && \
        pip install pycocotools

# Install maskrcnn
RUN git clone --single-branch --depth 1 https://github.com/matterport/Mask_RCNN.git $TEMP_MRCNN_DIR /tmp/maskrcnn && \
        cd /tmp/maskrcnn && \
        pip install -r requirements.txt && \
        python3 setup.py install

# Install DEXTR
RUN git clone --single --depth 1 https://github.com/jsbroks/dextr-keras.git /tmp/dextr && \
        cd /tmp/dextr && \
        pip install -r requirements.txt && \
        python setup.py install

COPY ./.git /workspace/.git
COPY ./app /workspace/app

COPY --from=build-stage /workspace/client/dist /workspace/dist

EXPOSE 5000
CMD gunicorn -w 1 -b 0.0.0.0:5000 app:app --worker-class eventlet --log-level info --no-sendfile --timeout 180


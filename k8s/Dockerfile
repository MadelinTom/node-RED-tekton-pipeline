FROM nodered/node-red
COPY package.json .
RUN npm install --unsafe-perm --no-update-notifier --no-fund --only=production
COPY flows.json /data/flows.json
COPY flows_cred.json /data/flows_cred.json
COPY settings.js .
EXPOSE 1880

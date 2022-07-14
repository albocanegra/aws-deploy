FROM node:18

ENV NVM_VERSION 9.11.2

# Install nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash && \
    export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")" && \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
    nvm install $NVM_VERSION

# Get instant-search-demo app
RUN mkdir /var/app && cd /var/app && \
    git clone https://github.com/algolia/instant-search-demo.git && \
    npm install ./instant-search-demo/

# Note: Deploying this image on ECS Fargate have resulted in the following error: 
# Error: ENOSPC: System limit for number of file watchers reached
# In order to avoid the node_modules directory to be "watched", the "npm install" command have been executed in the parent folder

WORKDIR /var/app/instant-search-demo

ENTRYPOINT npm start

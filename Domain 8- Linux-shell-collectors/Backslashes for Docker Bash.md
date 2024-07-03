#####  Backslashes
the command with the line-ending \ characters is meant for POSIX-compatible shells such as bash, not for cmd.exe

use \ for line-continuation (continuing a command on the following line) and escaping in general.
On windows, instead of using backslash (), you have to use backtick (`)

``````sh
docker run -e APP_NAME=strapi-app \
           -e DATABASE_CLIENT=mongo \
           -e DATABASE_HOST=strapi-mongo \
           -e DATABASE_PORT=27017 \
           -e DATABASE_NAME=strapi \
           -v `pwd`/strapi-app:/usr/src/api/strapi-app \
           --link strapi-mongo:mongo \
           -p 1337:1337 \
           --name strapi -d strapi/strapi
``````
#####  

``````sh
docker run -e APP_NAME=strapi-app `
           -e DATABASE_CLIENT=mongo `
           -e DATABASE_HOST=strapi-mongo `
           -e DATABASE_PORT=27017 `
           -e DATABASE_NAME=strapi `
           -v `pwd`/strapi-app:/usr/src/api/strapi-app `
           --link strapi-mongo:mongo `
           -p 1337:1337 `
           --name strapi -d strapi/strapi
``````

# Docker file build 
```sh
docker build -t learnereport_backend:latest .  
docker tag  learnereport_backend:latest  crashrik/learnereportcs_backend:latest 
docker push crashrik/learnereportcs_backend:latest
docker build -t learnereport_frontend:latest .  
```
# Docker run file 
```sh
docker run -d --env-file .env -p 3000:3000 crashrik/learnereportcs_backend:latest
docker run -it --env-file .env -p 80:3000 learnereport_frontend:latest
```
# troubleshoot 
```sh 
docker build --no-cache --progress=plain -t your-frontend .
```
## error
```
#8 [4/5] RUN npm install --silent
#8 3.361 npm notice 
#8 3.361 npm notice New major version of npm available! 8.19.4 -> 11.4.2
#8 3.362 npm notice Changelog: <https://github.com/npm/cli/releases/tag/v11.4.2>
#8 3.362 npm notice Run `npm install -g npm@11.4.2` to update!
#8 3.364 npm notice
#8 ERROR: process "/bin/sh -c npm install --silent" did not complete successfully: exit code: 1
------
 > [4/5] RUN npm install --silent:
 notice New major version of npm available! 8.19.4 -> 11.4.2
3.362 npm notice Changelog: <https://github.com/npm/cli/releases/tag/v11.4.2>
3.362 npm notice Run `npm install -g npm@11.4.2` to update!
3.364 npm notice
------

```
# Spleeter-Web UNRAID YML

This YML can help run https://github.com/JeffreyCA/spleeter-web on UNRAID.  Currently testing with in UNRAID using Portainer-CE to deploy the containers using this YML file.

As of 10/28/2021 you need to modify settings.py & settings_docker.py for api & both celery containers to use the local storage of Unraid. Copy files out of container > change permissions > modify DEFAULT_FILE_STORAGE > save > copy both files into all 3 containers > restart all containers.

Update:  v3.10 fixed the default_file_storage issue

A custom nginx.conf will need to be pushed to the nginx container to adjust the port it listens on and possibly to add another upstream server that references the api container's hostname (until I can figure out why API doesn't work all the time)

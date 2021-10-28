# Spleeter-Web UNRAID YML

This YML can help run https://github.com/JeffreyCA/spleeter-web on UNRAID.  Currently testing with in UNRAID using Portainer-CE to deploy the containers using this YML file.

As of 10/28/2021 you need to modify settings.py & settings_docker.py for api & both celery containers to use the local storage of Unraid. Copy files out of container > change permissions > modify DEFAULT_FILE_STORAGE > save > copy both files into all 3 containers > restart all containers.

# Spleeter-Web UNRAID YML

#Modify settings.py & settings_docker.py on api/celery_slow/celery_fast containers
#- DEFAULT_FILE_STORAGE=api.storage.FileSystemStorage
#Action items
#CPU Pinning / UID / GID / GIDLIST / UMASK / SLAVE:RW for unassigned drives

version: '3.4'
x-celery-env: &celery-env
  - DJANGO_SETTINGS_MODULE=django_react.settings_docker
  - CELERY_BROKER_URL=redis://redis:6379/0
  - CELERY_RESULT_BACKEND=redis://redis:6379/0
  - CPU_SEPARATION=0
  - CELERY_FAST_QUEUE_CONCURRENCY=${CELERY_FAST_QUEUE_CONCURRENCY:-3}
  - CELERY_SLOW_QUEUE_CONCURRENCY=${CELERY_SLOW_QUEUE_CONCURRENCY:-1}
  #- NVIDIA_VISIBLE_DEVICES=all #default
  - NVIDIA_VISIBLE_DEVICES=GPU-GUID #unraid GPU identifier to specify specific GPU
  - NVIDIA_DRIVER_CAPABILITIES=all
  
services:
  redis:
    image: redis:6.0-buster
    hostname: redis
    expose:
      - "6379"
    volumes:
      - redis-data:/data
    #custom unraid nw name
    networks:
      - dockernet
    restart: always
  #Removed since UNRAID runs nginx & port 80 conflicts with UNRAID
  api:
    image: jeffreyca/spleeter-web-backend:${TAG:-latest}-gpu
    #added ports
    ports:
      - "8000:8000"
    volumes:
      - assets:/webapp/frontend/assets
      - sqlite-data:/webapp/sqlite
      - staticfiles:/webapp/staticfiles
      #added volumes for self hosting & unraid shares
      - /mnt/user/unraid/paths:/webapp/media
    #custom unraid nw name
    networks:
      - dockernet
    stdin_open: true
    tty: true
    environment:
      - DJANGO_SETTINGS_MODULE=django_react.settings_docker
      - CELERY_BROKER_URL=redis://redis:6379/0
      - CELERY_RESULT_BACKEND=redis://redis:6379/0
      - CPU_SEPARATION=0
      #unraid host ip / name required?
      - APP_HOST=0.0.0.0
      #required?
      - YOUTUBE_API_KEY=
    depends_on:
      - redis
      - frontend
    restart: always
  celery-fast:
    image: jeffreyca/spleeter-web-backend:${TAG:-latest}-gpu
    entrypoint: ./celery-fast-entrypoint.sh
    volumes:
      - celery-data:/webapp/celery
      - pretrained-models:/webapp/pretrained_models
      - sqlite-data:/webapp/sqlite
      #added volumes for self hosting & unraid shares
      - /mnt/user/unraid/paths:/webapp/media
    #custom unraid nw name
    networks:
      - dockernet
    runtime: nvidia
    environment: *celery-env
    depends_on:
      - redis
    restart: always
  celery-slow:
    image: jeffreyca/spleeter-web-backend:${TAG:-latest}-gpu
    entrypoint: ./celery-slow-entrypoint.sh
    volumes:
      - celery-data:/webapp/celery
      - pretrained-models:/webapp/pretrained_models
      - sqlite-data:/webapp/sqlite
      #added volumes for self hosting & unraid shares
      - /mnt/user/unraid/paths:/webapp/media
    #custom unraid nw name
    networks:
      - dockernet
    runtime: nvidia
    environment: *celery-env
    depends_on:
      - redis
    restart: always
  frontend:
    image: jeffreyca/spleeter-web-frontend:${TAG:-latest}
    volumes:
      - assets:/webapp/frontend/assets
    #custom unraid nw name
    networks:
      - dockernet
    stdin_open: true
    tty: true
    restart: "no"
volumes:
  assets:
  celery-data:
  pretrained-models:
  redis-data:
  sqlite-data:
  staticfiles:
#custom unraid nw name
networks:
  dockernet:
    external: true

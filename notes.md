docker run --env-file .env -p 8000:8000 wanderlust_backend:latest
how test locally 


mongodb://${MONGO_INITDB_ROOT_USERNAME}:${MONGO_INITDB_ROOT_PASSWORD}@mongo:27017/wanderlust?authSource=admin
redis://:${REDIS_PASSWORD}@redis:6379


bhai jan donot change  variables name for dtabse it wasted my whole day keep them same like MONGO_INITDB_ROOT_PASSWORD  etc
## How to run containers via docker compose:
1. To run container with docker compose you need to create docker-compose.yml file
2. Then you need to populate the following code to that file:
version: '3'
networks:
  db_data_network:
    driver: bridge
services:
  mysql:
    image: mysql-local:1.0.0
    build:
      context: .
      dockerfile: Dockerfile.mysql
    container_name: mysql-container
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=1234
      - MYSQL_DATABASE=app_db
      - MYSQL_USER=app_user
      - MYSQL_PASSWORD=1234
    networks:
      - db_data_network
    healthcheck:
      test:
        [
          "CMD",
          "mysqladmin",
          "ping",
          "-h",
          "localhost",
          "-u",
          "app_user",
          "-p 1234"
        ]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
  todoapp:
    image: todoapp:1.0.0
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: todoapp-container
    ports:
      - "8080:8080"
    environment:
      - PYTHONUNBUFFERED=1
    networks:
      - db_data_network
    depends_on:
      mysql:
        condition: service_healthy
    restart: unless-stopped
volumes:
  mysql_data:
    driver: local
3. Then, you need to remove RUN python manage.py migrate as the database is no longer available at the build time
4. Then you need to refactor ENTRYPOINT to this: ENTRYPOINT ["sh", "-c", "python manage.py migrate && python manage.py runserver 0.0.0.0:8080"]
5. Then you need to replace "HOST" via you container/service name because they are in the same network
6. To run docker compose run command: docker-compose up -d
7. To stop docker compose run command: docker-compose down
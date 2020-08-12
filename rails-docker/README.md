# WIP

## Workflow

### Initial setup

1. Create project foler
```
mkdir dockerized-rails-app
cd dockerized-rails-app
touch Dockerfile
```

2. Open Dockerfile and add the following line
```
FROM starefossen/ruby-node:2-10
RUN apt-get update -qq && \
    apt-get install -y nano build-essential libpq-dev && \
    gem install bundler
RUN mkdir /project
COPY Gemfile Gemfile.lock /project/
WORKDIR /project
RUN bundle install
COPY . /project
```

This image contains node as well as ruby.

3. Create `docker-compose.yml` file in **project root folder**
```
version: '3.2'
volumes:
  postgres-data:
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: 'password'
    volumes:
      - postgres-data:/var/lib/postgresql/data
  app:
    build:
      context: .
      dockerfile: Dockerfile
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
      - .:/project
    ports:
      - "3000:3000"
    depends_on:
      - db
```

4. Add empty `Gemfile` and `Gemfile.lock` in **project root folder**. Add the following to lines to `Gemfile` file

```
source 'https://rubygems.org'
gem 'rails', '~> 6.0', '>= 6.0.3.2'
```

5. Run following command in **project root folder**

```
docker-compose build
```

6. Create new project by running the following command in **project root folder**
```
docker-compose run --rm --user $(id -u):$(id -g) app rails new . --force --database=postgresql --skip-bundle
```

We are skipping bundle install on purpose. When a new project is generated the Gemfile on our system will be updated with the Rails dependendcies and we have to rebuild our image.

7. Next update Rails application to use the postgres from our services file:

```
# config/database.yml

# ...
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  user: postgres
  port: 5432
  password:
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
# ... leave all else intact
```


8. Run `docker-compose build app` in **project root folder**

9. Run `docker-compose run --rm app rails webpacker:install` in **project root folder**

9. Run `docker-compose run --rm app rails db:create` in **project root folder**

## References

https://auth0.com/blog/ruby-on-rails-killer-workflow-with-docker-part-1/
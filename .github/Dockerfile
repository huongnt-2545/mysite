FROM ruby:2.7.8-bullseye AS devel

RUN apt-get update && \
    apt-get install -y \
        default-libmysqlclient-dev \
        shared-mime-info \
        nodejs \
        npm \
        tzdata \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g yarn

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN bundle install --jobs 4 --retry 3

COPY package.json yarn.lock ./

RUN NODE_ENV=development yarn install


FROM devel AS builder

ENV BUNDLE_WITHOUT="development test"
ENV RAILS_ENV=production

COPY . .

RUN rails assets:precompile
RUN bundle clean --force

FROM ruby:2.7.8-slim-bullseye

ENV RAILS_LOG_TO_STDOUT=true

RUN apt-get update && \
    apt-get install -y \
        default-libmysqlclient-dev

WORKDIR /app

COPY --from=builder /usr/local/bundle/ /usr/local/bundle/

COPY Gemfile Gemfile.lock ./

COPY . .

COPY --from=builder /app/public/assets /app/public/assets
COPY --from=builder /app/public/packs /app/public/packs

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]

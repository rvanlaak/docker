FROM webdevops/php-apache-dev:7.3

RUN	apt-get update && apt-get install -y \
		zlib1g-dev \
		chromium-driver \
&& rm -rf /var/lib/apt/lists/*

ENV PANTHER_NO_SANDBOX 1
ENV PANTHER_CHROME_DRIVER_BINARY /usr/bin/chromedriver

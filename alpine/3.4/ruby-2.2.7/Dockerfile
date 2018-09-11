FROM alpine:3.4
LABEL MAINTAINER "Lucas Montiel <montiel@alfaconcursos.com.br>"
RUN mkdir -p /usr/local/etc \
	&& { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.7
ENV RUBY_DOWNLOAD_SHA256 234c8aee6543da9efd67008e6e7ee740d41ed57a52e797f65043c3b5ec3bcb53
ENV RUBYGEMS_VERSION 2.6.13
ENV CONFIGURE_OPTS=--disable-install-doc

RUN set -ex \
	\
	&& apk add --update --no-cache --virtual .ruby-builddeps \
		autoconf \
		bison \
		bzip2 \
		bzip2-dev \
		ca-certificates \
		coreutils \
		dpkg-dev dpkg \
		file \
		gcc \
		gdbm-dev \
		glib-dev \
		imagemagick \
		libc-dev \
		libffi-dev \
		openssh \
		openssl \
		openssl-dev \
		libxml2-dev \
		libxslt-dev \
		linux-headers \
		make \
		mysql-dev \
		nodejs \
		ncurses-dev \
		postgresql-dev \
		procps \
		qt-dev \
		readline-dev \
		ruby \
		ruby-json \
		sqlite-dev \
		tar \
		tzdata \
		xz \
		yaml-dev \
		zlib-dev \
	\
	&& wget -O ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz" \
	&& echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum -c - \
	\
	&& mkdir -p /usr/src/ruby \
	&& tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1 \
	&& rm ruby.tar.xz \
	\
	&& cd /usr/src/ruby \
	\

	&& { \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new \
	&& mv file.c.new file.c \
	\
	&& autoconf \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& export ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
	&& ./configure \
		--build="$gnuArch" \
		--disable-install-doc \
		--enable-shared \
	&& make -j "$(nproc)" \
	&& make install \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner --recursive /usr/local \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
	&& apk add --virtual .ruby-rundeps $runDeps \
		bzip2 \
		ca-certificates \
		libffi-dev \
		openssl-dev \
		procps \
		yaml-dev \
		zlib-dev \
	&& apk del .ruby-builddeps \
	&& cd / \
	&& rm -r /usr/src/ruby \
	\
	&& gem update --system "$RUBYGEMS_VERSION"

ENV BUNDLER_VERSION="1.14.6"

RUN gem install bundler --version "$BUNDLER_VERSION"

ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
	BUNDLE_BIN="$GEM_HOME/bin" \
	BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
	&& chmod 777 "$GEM_HOME" "$BUNDLE_BIN"

ADD ./wrapdocker /usr/local/bin/wrapdocker

RUN chmod +x /usr/local/bin/wrapdocker && \
    echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/main" >> /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories && \
    apk add --upgrade --no-cache bash \
		       curl \
		       docker=18.06.1-r0 \
		       python \
		       python-dev \
        	   py-pip \
        	   yarn && \
    pip install --upgrade pip && \
    pip install awscli && \
    apk --purge -v del py-pip
VOLUME /var/lib/docker
CMD ["./usr/local/bin/wrapdocker"]
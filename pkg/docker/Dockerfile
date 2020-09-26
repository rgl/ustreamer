FROM raspbian:buster-slim as build
RUN echo 'deb http://archive.raspberrypi.org/debian/ buster main' >/etc/apt/sources.list.d/raspi.list && \
	gpg --keyserver keys.gnupg.net --recv-key 82B129927FA3303E && \
	gpg -a --export 82B129927FA3303E | apt-key add - && \
	apt-get update && \
	apt-get install -y --no-install-recommends \
		gcc \
		make \
		libbsd-dev \
		libevent-dev \
		libgpiod-dev \
		libjpeg8-dev \
		libraspberrypi-dev \
		uuid-dev \
		&& \
	rm -rf /var/lib/apt/lists/*
WORKDIR /build/ustreamer/
COPY . .
RUN make -j5 WITH_OMX=1 WITH_GPIO=1

FROM raspbian:buster-slim
RUN echo 'deb http://archive.raspberrypi.org/debian/ buster main' >/etc/apt/sources.list.d/raspi.list && \
	gpg --keyserver keys.gnupg.net --recv-key 82B129927FA3303E && \
	gpg -a --export 82B129927FA3303E | apt-key add - && \
	apt-get update && \
	apt-get install -y --no-install-recommends \
		libbsd0 \
		libevent-2.1 \
		libevent-pthreads-2.1-6 \
		libgpiod2 \
		libjpeg8 \
		libraspberrypi0 \
		libuuid1 \
		&& \
	rm -rf /var/lib/apt/lists/*
WORKDIR /ustreamer
COPY --from=build /build/ustreamer/ustreamer .
EXPOSE 8080
ENTRYPOINT ["./ustreamer"]

# vim: syntax=dockerfile

FROM debian:bullseye
MAINTAINER ccondit@randomcoder.org

ADD assets /assets

RUN \
	echo "deb http://ftp.us.debian.org/debian bullseye main non-free contrib" > /etc/apt/sources.list && \
        echo "deb http://security.debian.org/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list && \
	echo "deb-src http://ftp.us.debian.org/debian sid main contrib non-free" >> /etc/apt/sources.list && \
	apt-get update && \
	apt-get -y install gnupg2 && \
	apt-key add /assets/bintray-public.key.asc && \
	apt-get update && \
	apt-get -y dist-upgrade && \
	apt-get -y install dpkg-dev build-essential devscripts lintian patch patchutils && \
	rm -rf /var/cache/apt && \
	mkdir -p /build /packages

RUN \
	apt-get update && \
	apt-get -y install dh-python python-all-dev python3-all-dev python-all-dbg python3-all-dbg

ADD patches /patches

RUN \
	apt-get update && \
	apt-get -y install python3-stdeb python3-setuptools && \
	cd /build && \
        curl -O \
                'https://files.pythonhosted.org/packages/bd/ca/204b2d13bfcaac1c7be6e76fc473a8a339570734884b952b5440e853a7ea/certbot-external-auth-0.1.2.tar.gz' && \
	py2dsc \
                --with-python3=True --with-python2=False \
		--suite bullseye-backports-randomcoder \
		--maintainer 'Craig Condit <ccondit@randomcoder.org>' \
		certbot-external-auth-0.1.2.tar.gz && \
	cd deb_dist && \
	cd certbot-external-auth-* && \
	export DEBEMAIL=ccondit@randomcoder.org && \
	export DEBFULLNAME="Craig Condit" && \
	dch -b \
		-l "+local" \
		--distribution bullseye-backports-randomcoder \
		--force-distribution --noquery \
		'Build for Debian Jessie.' && \
	patch -p1 < /patches/allow-noninteractive.patch && \
	dch -b \
		--distribution bullseye-backports-randomcoder \
		--force-distribution --noquery \
		--increment \
		'Allow non-interactive usage.' && \
	debian/rules binary && \
	cd .. && \
	cp -f *+local*.deb /packages/


ERR      = *** Unexpected MAKELEVEL = 0.
HINT     = This Makefile can't be used directly, consider cd ../.. before try again
$(if $(subst 0,,${MAKELEVEL}),,$(warning $(ERR));$(error $(HINT)))

V_ZLIB  = 1.2.11
V_SSL   = 1.1.1b
V_ARES  = 1.15.0
V_CURL  = 7.64.1
V_SQL   = 3270200
V_NCUR  = 6.1
V_JSON  = 3.6.1
V_CATCH = 2.7.0
V_UWS   = 0.14.8
V_QF    = 1.15.1
V_UV    = 1.27.0

KBUILD := $(abspath ../../build-$(KHOST))

all dist:
	@mkdir -vp $(KBUILD)
	$(MAKE) zlib openssl ares curl sqlite ncurses json catch uws quickfix libuv
ifdef TRAVIS_OS_NAME
	$(MAKE) pvs
endif

clean:
	@rm -vrf $(KBUILD)

zlib:
	test -d $(KBUILD)/zlib-$(V_ZLIB) || (                                             \
	curl -L https://zlib.net/zlib-$(V_ZLIB).tar.gz | tar xz -C $(KBUILD)              \
	&& cd $(KBUILD)/zlib-$(V_ZLIB) && (test -n "`echo $(CHOST) | grep mingw32`"       \
	&& DESTDIR=$(KBUILD)/local/ BINARY_PATH=bin INCLUDE_PATH=include LIBRARY_PATH=lib \
	make all install -fwin32/Makefile.gcc PREFIX=$(CHOST)-                            \
	|| (./configure --static --prefix=$(KBUILD)/local && make all install))           )

openssl:
	test -d $(KBUILD)/openssl-$(V_SSL) || (                                                   \
	curl -L https://www.openssl.org/source/openssl-$(V_SSL).tar.gz | tar xz -C $(KBUILD)      \
	&& cd $(KBUILD)/openssl-$(V_SSL) && CC=gcc                                                \
	./Configure $(shell test -n "`echo $(CHOST) | grep mingw32`" && echo mingw64 || echo gcc) \
	--cross-compile-prefix=$(CHOST)- --prefix=$(KBUILD)/local --openssldir=$(KBUILD)/local    \
	&& make all install_sw install_ssldirs                                                    )

ares:
	test -d $(KBUILD)/c-ares-$(V_ARES) || (                                                  \
	curl -L https://c-ares.haxx.se/download/c-ares-$(V_ARES).tar.gz | tar xz -C $(KBUILD)    \
	&& cd $(KBUILD)/c-ares-$(V_ARES) && ./configure --host=$(CHOST) --prefix=$(KBUILD)/local \
	--disable-shared --enable-static && make all install                                     )

curl:
	test -d $(KBUILD)/curl-$(V_CURL) || (                                                   \
	curl -L https://curl.haxx.se/download/curl-$(V_CURL).tar.gz | tar xz -C $(KBUILD)       \
	&& cd $(KBUILD)/curl-$(V_CURL)                                                          \
	&& CPPFLAGS=-DCARES_STATICLIB ./configure --host=$(CHOST) --prefix=$(KBUILD)/local      \
	--with-ca-bundle=/etc/ssl/certs/ca-certificates.crt                                     \
	--with-zlib=$(KBUILD)/local --with-ssl=$(KBUILD)/local --enable-ares=$(KBUILD)/local    \
	--disable-manual --disable-imap --disable-file --disable-tftp --disable-smtp            \
	--disable-pop3 --disable-smb --disable-ftp --disable-sspi --disable-ldap --disable-dict \
	--without-libpsl --without-nghttp2 --without-libssh2 --disable-telnet --disable-rtsp    \
	--disable-gopher --without-libidn2 --without-librtmp --disable-shared --enable-static   \
	&& sed -i "/HAVE_GETEUID/d" lib/curl_config.h && make all install                       )

sqlite:
	test -d $(KBUILD)/sqlite-autoconf-$(V_SQL) || (                                       \
	curl -L https://sqlite.org/2019/sqlite-autoconf-$(V_SQL).tar.gz | tar xz -C $(KBUILD) \
	&& cd $(KBUILD)/sqlite-autoconf-$(V_SQL) && ./configure --prefix=$(KBUILD)/local      \
	--host=$(CHOST) --enable-static --disable-dynamic-extensions --disable-shared         \
	--enable-threadsafe && make all install                                               )

ncurses:
	test -d $(KBUILD)/ncurses-$(V_NCUR) || (                                                          \
	curl -L http://ftp.gnu.org/pub/gnu/ncurses/ncurses-$(V_NCUR).tar.gz | tar xz -C $(KBUILD)         \
	&& cd $(KBUILD)/ncurses-$(V_NCUR) && CPPFLAGS=-P ./configure                                      \
	--host=$(CHOST) --prefix=$(KBUILD)/local $(shell test -n "`echo $(CHOST) | grep mingw32`" && echo \
	--without-cxx-binding --without-ada --enable-reentrant --with-normal                              \
	--disable-home-terminfo --enable-sp-funcs --enable-term-driver --enable-interop || :)             \
	--disable-lib-suffixes --without-debug --without-progs --without-tests                            \
	--with-fallbacks=linux,screen,vt100,xterm,xterm-256color,putty-256color && make all install       )

json:
	test -f $(KBUILD)/local/include/json.h || (mkdir -p $(KBUILD)/local/include       \
	&& curl -L https://github.com/nlohmann/json/releases/download/v$(V_JSON)/json.hpp \
	-o $(KBUILD)/local/include/json.h &&                                              \
	sed -i '$$iusing namespace nlohmann;' $(KBUILD)/local/include/json.h &&           \
	(test -n "`echo $(CHOST) | grep mingw32`" &&                                      \
	sed -i "s/\(#include <cstddef>\)/\/\/\1/" $(KBUILD)/local/include/json.h ||     :))

catch:
	test -f $(KBUILD)/local/include/catch.h || (mkdir -p $(KBUILD)/local/include          \
	&& curl -L https://github.com/catchorg/Catch2/releases/download/v$(V_CATCH)/catch.hpp \
	-o $(KBUILD)/local/include/catch.h                                                    )

uws:
	test -d $(KBUILD)/uWebSockets-$(V_UWS)                                                  \
	|| curl -L https://github.com/uNetworking/uWebSockets/archive/v$(V_UWS).tar.gz          \
	| tar xz -C $(KBUILD) && rm -rf $(KBUILD)/local/include/uWS                             \
	&& mkdir -p $(KBUILD)/local/include/uWS                                                 \
	&& cp $(KBUILD)/uWebSockets-$(V_UWS)/src/* $(KBUILD)/local/include/uWS/                 \
	&& sed -i "s/SSL_library_init()/OPENSSL_init_ssl(0, NULL)/"                             \
	$(KBUILD)/local/include/uWS/Networking.cpp && (test -n "`echo $(CHOST) | grep mingw32`" \
	&& (sed -i "s/W\(s2tcpip\)/w\1/" $(KBUILD)/local/include/uWS/Networking.h               \
	&& sed -i "s/WinSock2/winsock2/" $(KBUILD)/local/include/uWS/Networking.h               \
	&& sed -i "s/uv_.*/return {0, \"\", \"\"};/" $(KBUILD)/local/include/uWS/Socket.cpp     \
	&& sed -i "/sockaddr_storage/,+16d" $(KBUILD)/local/include/uWS/Socket.cpp) ||         :)

quickfix:
	test -d $(KBUILD)/quickfix-$(V_QF) || (                                                      \
	curl -L https://github.com/quickfix/quickfix/archive/v$(V_QF).tar.gz | tar xz -C $(KBUILD)   \
	&& cd $(KBUILD)/quickfix-$(V_QF) && ./bootstrap                                              \
	&& sed -i "s/bin spec test examples doc//" Makefile.am                                       \
	&& sed -i "s/SUBDIRS = test//" src/C++/Makefile.am                                           \
	&& sed -i "/Certificate Verification: depth/,+2d" src/C++/UtilitySSL.cpp                     \
	&& sed -i "s/throw.\?( .* )//" src/C++/*.h src/C++/*.cpp                                     \
	&& sed -i "/int r;/,+9d" src/C++/AtomicCount.h                                               \
	&& sed -i "s/\/\/ \(\*\)/\1/" src/C++/AtomicCount.h                                          \
	&& sed -i "s/\/\/ \(i\)/\1/" src/C++/AtomicCount.h                                           \
	&& (test -n "`echo $(CHOST) | grep mingw32`"                                                 \
	&& (sed -i "s/Thread/ThreadId/" src/C++/Utility*.cpp                                         \
	&& sed -i "s/_MSC_VER/_WIN32/" src/C++/Utility*.* src/C++/SSLSocketInitiator.h               \
	&& sed -i "s/stdafx/config/" src/C++/Utility*.cpp src/C++/SSLSocketInitiator.cpp             \
	&& sed -i "s/socklen_t/int/" src/C++/Utility.cpp                                             \
	&& sed -i "s/INVALID_SOCKET/~0/" src/C++/Utility.cpp                                         \
	&& sed -i "/host = socket_hostname/d" src/C++/Utility.cpp                                    \
	&& sed -i "s/_stdcall\( \*\)*//" src/C++/Utility.h                                           \
	&& sed -i "s/\(typedef int\)/\/\/\1/" src/C++/Utility.h src/C++/SocketMonitor.h              \
	&& sed -i "s/W\(insock2\)/w\1/" src/C++/Utility.h src/C++/SocketMonitor.h                    \
	&& sed -i "/pragma/d" src/C++/Utility.h src/C++/SocketMonitor.h src/C++/SSLSocketInitiator.h \
	&& sed -i "s/\(ifdef\) _MSC_VER/\1 _WIN32/" src/C++/SocketMonitor.h src/C++/Mutex.h          \
	&& sed -i "s/\(ifndef\) _MSC_VER/\1 _WIN32/" src/C++/SocketConnector.cpp                     \
	src/C++/SocketServer.cpp) || :) &&                                                           \
	./configure --prefix=$(KBUILD)/local --host=$(CHOST) --with-openssl=$(KBUILD)/local          \
	--enable-shared=no --enable-static=yes && cd src/C++ && make all install                     )

libuv:
	test -z "`echo $(CHOST) | grep darwin;echo $(CHOST) | grep mingw32`" || test -d $(KBUILD)/libuv-$(V_UV) || ( \
	curl -L https://github.com/libuv/libuv/archive/v$(V_UV).tar.gz | tar xz -C $(KBUILD)                         \
	&& cd $(KBUILD)/libuv-$(V_UV) && sh autogen.sh && CC=$(CHOST)-clang ./configure --host=$(CHOST)              \
	--prefix=$(KBUILD)/local && make all install                                                                 )

pvs:
ifndef V_PVS
	$(MAKE) $@ V_PVS=$(shell curl -s https://www.viva64.com/en/pvs-studio-download/ | grep x86_64.tgz | sed 's/.*href=\"\(.*\)\" .*/\1/' | cut -d '-' -f3)
else
	test -d $(KBUILD)/pvs-studio-$(V_PVS)-x86_64 || (                \
	curl -L http://files.viva64.com/pvs-studio-$(V_PVS)-x86_64.tgz   \
	| tar xz -C $(KBUILD) && cd $(KBUILD)/pvs-studio-$(V_PVS)-x86_64 \
	&& chmod +x install.sh && sudo ./install.sh                      )
endif

.PHONY: all dist clean zlib openssl curl sqlite ncurses json catch uws quickfix libuv pvs
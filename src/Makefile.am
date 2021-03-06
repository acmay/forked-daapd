
sbin_PROGRAMS = forked-daapd

if COND_ITUNES
ITUNES_SRC=filescanner_itunes.c
endif

if COND_SPOTIFY
SPOTIFY_SRC=spotify.c spotify.h
endif

if COND_LASTFM
LASTFM_SRC=lastfm.c lastfm.h
endif

if COND_CHROMECAST
if COND_PROTOBUF_OLD
CHROMECAST_SRC=outputs/cast.c outputs/cast_channel.v0.pb-c.h outputs/cast_channel.v0.pb-c.c
else
CHROMECAST_SRC=outputs/cast.c outputs/cast_channel.pb-c.h outputs/cast_channel.pb-c.c
endif
endif

if COND_MPD
MPD_SRC=mpd.c mpd.h
endif

if COND_ALSA
ALSA_SRC=outputs/alsa.c
endif

if COND_PULSEAUDIO
PULSEAUDIO_SRC=outputs/pulse.c
endif

GPERF_FILES = \
	daap_query.gperf \
	rsp_query.gperf \
	dacp_prop.gperf \
	dmap_fields.gperf

GPERF_PRODUCTS = \
	daap_query_hash.c \
	rsp_query_hash.c \
	dacp_prop_hash.c \
	dmap_fields_hash.c

ANTLR_GRAMMARS = \
	RSP.g RSP2SQL.g \
	DAAP.g DAAP2SQL.g \
	SMARTPL.g SMARTPL2SQL.g 

ANTLR_SOURCES = \
	RSPLexer.c RSPLexer.h RSPParser.c RSPParser.h \
	RSP2SQL.c RSP2SQL.h \
	DAAPLexer.c DAAPLexer.h DAAPParser.c DAAPParser.h \
	DAAP2SQL.c DAAP2SQL.h \
	SMARTPLLexer.c SMARTPLLexer.h SMARTPLParser.c SMARTPLParser.h  \
	SMARTPL2SQL.c SMARTPL2SQL.h

ANTLR_PRODUCTS =

forked_daapd_CPPFLAGS = -D_GNU_SOURCE \
	-DDATADIR="\"$(pkgdatadir)\"" -DCONFDIR="\"$(sysconfdir)\"" \
	-DSTATEDIR="\"$(localstatedir)\"" -DPKGLIBDIR="\"$(pkglibdir)\""

forked_daapd_CFLAGS = \
	@ZLIB_CFLAGS@ @AVAHI_CFLAGS@ @SQLITE3_CFLAGS@ @LIBAV_CFLAGS@ \
	@CONFUSE_CFLAGS@ @MINIXML_CFLAGS@ @LIBPLIST_CFLAGS@ @SPOTIFY_CFLAGS@ \
	@LIBGCRYPT_CFLAGS@ @GPG_ERROR_CFLAGS@ @ALSA_CFLAGS@ @LIBPULSE_CFLAGS@ \
	@LIBCURL_CFLAGS@ @LIBPROTOBUF_C_CFLAGS@ @GNUTLS_CFLAGS@ @JSON_C_CFLAGS@

forked_daapd_LDADD = -lrt \
	@ZLIB_LIBS@ @AVAHI_LIBS@ @SQLITE3_LIBS@ @LIBAV_LIBS@ \
	@CONFUSE_LIBS@ @LIBEVENT_LIBS@ @LIBUNISTRING@ \
	@MINIXML_LIBS@ @ANTLR3C_LIBS@ @LIBPLIST_LIBS@ @SPOTIFY_LIBS@ \
	@LIBGCRYPT_LIBS@ @GPG_ERROR_LIBS@ @ALSA_LIBS@ @LIBPULSE_LIBS@ \
	@LIBCURL_LIBS@ @LIBPROTOBUF_C_LIBS@ @GNUTLS_LIBS@ @JSON_C_LIBS@

forked_daapd_SOURCES = main.c \
	db.c db.h \
	db_init.c db_init.h \
	db_upgrade.c db_upgrade.h \
	logger.c logger.h \
	conffile.c conffile.h \
	cache.c cache.h \
	filescanner.c filescanner.h \
	filescanner_ffmpeg.c filescanner_playlist.c \
	filescanner_smartpl.c $(ITUNES_SRC) \
	mdns_avahi.c mdns.h \
	remote_pairing.c remote_pairing.h \
	avio_evbuffer.c avio_evbuffer.h \
	httpd.c httpd.h \
	httpd_rsp.c httpd_rsp.h \
	httpd_daap.c httpd_daap.h \
	httpd_dacp.c httpd_dacp.h \
	httpd_streaming.c httpd_streaming.h \
	http.c http.h \
	dmap_common.c dmap_common.h \
	transcode.c transcode.h \
	pipe.c pipe.h \
	artwork.c artwork.h \
	misc.c misc.h \
	rng.c rng.h \
	rsp_query.c rsp_query.h \
	daap_query.c daap_query.h \
	player.c player.h \
	queue.c queue.h \
	worker.c worker.h \
	outputs.h outputs.c \
	outputs/raop.c outputs/streaming.c outputs/dummy.c outputs/fifo.c \
	$(ALSA_SRC) $(PULSEAUDIO_SRC) $(CHROMECAST_SRC) \
	evrtsp/rtsp.c evrtp/evrtsp.h evrtsp/rtsp-internal.h evrtsp/log.h \
	$(SPOTIFY_SRC) \
	$(LASTFM_SRC) \
	$(MPD_SRC) \
	listener.c listener.h \
	commands.c commands.h

nodist_forked_daapd_SOURCES = \
	$(ANTLR_SOURCES)

BUILT_SOURCES = \
	$(GPERF_PRODUCTS)

EXTRA_DIST = \
	$(ANTLR_GRAMMARS)

CLEANFILES = \
	$(GPERF_PRODUCTS)


# gperf construction rules
%_hash.c: %.gperf
	if $(GPERF) $< > $@.tmp; then \
		mv $@.tmp $@; \
	elif $(GPERF) --version >/dev/null 2>&1; then \
		rm $@.tmp; \
		exit 1; \
	else \
		rm $@.tmp; \
		touch $@; \
	fi

# Support for building the parsers when ANTLR3 is available
if COND_ANTLR
SUFFIXES = .g .u

%.tokens %.c %Lexer.c %Parser.c %Lexer.h %Parser.h %.h: %.g
	$(ANTLR) -Xconversiontimeout 30000 $(ANTLR_OPTIONS) $<

%.u: %.g
	$(ANTLR) -depend $< > $@
	@echo -n "ANTLR_PRODUCTS += " > $@.tmp
	@grep : $@ | cut -d : -f 1 | tr -d ' ' | { while read f; do test "$$f" != "$<" && echo -n "$$f "; done } >> $@.tmp
	@cat $@.tmp >> $@
	@rm $@.tmp

BUILT_SOURCES += $(ANTLR_SOURCES)

CLEANFILES += \
	$(ANTLR_PRODUCTS) \
	$(ANTLR_GRAMMARS:.g=.u)

else !COND_ANTLR
DISTCLEANFILES = \
	$(ANTLR_PRODUCTS) \
	$(ANTLR_GRAMMARS:.g=.u)

endif

-include $(ANTLR_GRAMMARS:.g=.u)

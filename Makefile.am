ACLOCAL_AMFLAGS = -I m4

# FIXME: These flags should be produced in configure.ac using AX_APPEND_COMPILE_FLAGS
AM_CFLAGS = \
	-Wall -Wstrict-prototypes -Wmissing-prototypes -Wmissing-declarations \
	-Wunused -fstrict-aliasing -Wextra -Wformat=2 -Winit-self -Winline \
	-Wpacked -Wpointer-arith -Wmissing-format-attribute -Wmissing-noreturn \
	-Wnested-externs -Wstrict-prototypes -Wunsafe-loop-optimizations \
	-Wwrite-strings -Wno-missing-field-initializers -Wno-unused-parameter \
	-Wcast-align -Wformat-nonliteral -Wformat-security -Wswitch-default \
	-Wmissing-include-dirs -Waggregate-return -Wunused-but-set-variable \
	-Warray-bounds -Wold-style-definition -Wsign-compare -Wlarger-than=65537
AM_CFLAGS += -Wno-undef  # sophiasip is not -Wundef--safe
AM_CFLAGS += -Wno-redundant-decls  # sophiasip also contains redundant declarations
AM_CFLAGS += -Wno-override-init  # we need this for JANUS_PLUGIN_INIT
# FIXME: These should be enabled once the code is safe for them. That requires
# some fairly big refactoring though, which can wait.
# AM_CFLAGS += -Wshadow -Wstrict-aliasing=2

AM_CFLAGS += -fstack-protector-all -g -ggdb -fPIC -rdynamic -pthread

# FIXME: make docs work with distcheck
DISTCHECK_CONFIGURE_FLAGS = --disable-docs --enable-post-processing

EXTRA_DIST = $(NULL)
CLEANFILES = $(NULL)

bin_PROGRAMS = janus

headerdir = $(includedir)/janus
header_HEADERS = \
	plugins/plugin.h apierror.h config.h debug.h dtls.h ice.h mutex.h record.h \
	rtcp.h rtp.h sctp.h sdp.h utils.h

confdir = $(sysconfdir)/janus
conf_DATA = conf/janus.cfg.sample

certdir = $(datadir)/janus/certs
cert_DATA = certs/mycert.key certs/mycert.pem

plugindir = $(libdir)/janus/plugins
plugin_LTLIBRARIES = $(NULL)

SUBDIRS = html
html_DATA = README.md

streamdir = $(datadir)/janus/streams
stream_DATA = $(NULL)

recordingsdir = $(datadir)/janus/recordings
recordings_DATA = $(NULL)

%.sample: %.sample.in
	$(MKDIR_P) $(@D)
	$(AM_V_GEN) sed -e "\
	    s|[@]confdir[@]|$(confdir)|;\
	    s|[@]certdir[@]|$(certdir)|;\
	    s|[@]plugindir[@]|$(plugindir)|;\
	    s|[@]recordingsdir[@]|$(recordingsdir)|;\
	    s|[@]streamdir[@]|$(streamdir)|" \
	$< > $@ || rm $@

EXTRA_DIST += conf/janus.cfg.sample.in
CLEANFILES += conf/janus.cfg.sample

##
# Janus
##

janus_SOURCES = \
	apierror.c \
	apierror.h \
	cmdline.c \
	cmdline.h \
	config.c \
	config.h \
	debug.h \
	dtls.c \
	dtls.h \
	ice.c \
	ice.h \
	janus.c \
	janus.h \
	mutex.h \
	record.c \
	record.h \
	rtcp.c \
	rtcp.h \
	rtp.h \
	sctp.c \
	sctp.h \
	sdp.c \
	sdp.h \
	utils.c \
	utils.h \
	plugins/plugin.c \
	plugins/plugin.h \
	$(NULL)

janus_CFLAGS = \
	$(AM_CFLAGS) \
	$(JANUS_CFLAGS) \
	-DPLUGINDIR=\"$(plugindir)\" \
	-DCONFDIR=\"$(confdir)\" \
	$(NULL)

janus_LDADD = \
	$(JANUS_LIBS) \
	$(JANUS_MANUAL_LIBS) \
	-lsrtp \
	$(NULL)

BUILT_SOURCES = cmdline.c cmdline.h

cmdline.c: janus.ggo
	gengetopt --set-package="janus" --set-version="$(VERSION)" < $^

EXTRA_DIST += janus.ggo
CLEANFILES += cmdline.c cmdline.h

##
# Plugins
##

plugins_cflags = \
	$(AM_CFLAGS) \
	$(PLUGINS_CFLAGS) \
	$(NULL)

plugins_libadd = \
	$(PLUGINS_LIBS) \
	$(NULL)

if ENABLE_PLUGIN_AUDIOBRIDGE
plugin_LTLIBRARIES += plugins/libjanus_audiobridge.la
plugins_libjanus_audiobridge_la_SOURCES = plugins/janus_audiobridge.c
plugins_libjanus_audiobridge_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_audiobridge_la_LDFLAGS = $(plugins_ldflags) -lopus
plugins_libjanus_audiobridge_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.audiobridge.cfg.sample
EXTRA_DIST += conf/janus.plugin.audiobridge.cfg.sample
endif

if ENABLE_PLUGIN_ECHOTEST
plugin_LTLIBRARIES += plugins/libjanus_echotest.la
plugins_libjanus_echotest_la_SOURCES = plugins/janus_echotest.c
plugins_libjanus_echotest_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_echotest_la_LDFLAGS = $(plugins_ldflags)
plugins_libjanus_echotest_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.echotest.cfg.sample
EXTRA_DIST += conf/janus.plugin.echotest.cfg.sample
endif

if ENABLE_PLUGIN_RECORDPLAY
plugin_LTLIBRARIES += plugins/libjanus_recordplay.la
plugins_libjanus_recordplay_la_SOURCES = plugins/janus_recordplay.c
plugins_libjanus_recordplay_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_recordplay_la_LDFLAGS = $(plugins_ldflags)
plugins_libjanus_recordplay_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.recordplay.cfg.sample
recordings_DATA += \
	plugins/recordings/1234.nfo \
	plugins/recordings/rec-sample-audio.mjr \
	plugins/recordings/rec-sample-video.mjr
EXTRA_DIST += \
	conf/janus.plugin.recordplay.cfg.sample.in \
	$(recordings_DATA)
CLEANFILES += conf/janus.plugin.recordplay.cfg.sample
endif

if ENABLE_PLUGIN_SIP
plugin_LTLIBRARIES += plugins/libjanus_sip.la
plugins_libjanus_sip_la_SOURCES = plugins/janus_sip.c
plugins_libjanus_sip_la_CFLAGS = $(plugins_cflags) $(PLUGIN_SIP_CFLAGS)
plugins_libjanus_sip_la_LDFLAGS = $(plugins_ldflags) $(PLUGIN_SIP_LDFLAGS)
plugins_libjanus_sip_la_LIBADD = $(plugins_libadd) $(PLUGIN_SIP_LIBADD)
conf_DATA += conf/janus.plugin.sip.cfg.sample
EXTRA_DIST += conf/janus.plugin.sip.cfg.sample
endif

if ENABLE_PLUGIN_STREAMING
plugin_LTLIBRARIES += plugins/libjanus_streaming.la
plugins_libjanus_streaming_la_SOURCES = plugins/janus_streaming.c
plugins_libjanus_streaming_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_streaming_la_LDFLAGS = $(plugins_ldflags)
plugins_libjanus_streaming_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.streaming.cfg.sample
stream_DATA += \
	plugins/streams/music.mulaw \
	plugins/streams/radio.alaw \
	plugins/streams/test_gstreamer.sh \
	plugins/streams/test_gstreamer_1.sh
EXTRA_DIST += \
	conf/janus.plugin.streaming.cfg.sample.in \
	$(stream_DATA)
CLEANFILES += conf/janus.plugin.streaming.cfg.sample
endif

if ENABLE_PLUGIN_VIDEOCALL
plugin_LTLIBRARIES += plugins/libjanus_videocall.la
plugins_libjanus_videocall_la_SOURCES = plugins/janus_videocall.c
plugins_libjanus_videocall_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_videocall_la_LDFLAGS = $(plugins_ldflags)
plugins_libjanus_videocall_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.videocall.cfg.sample
EXTRA_DIST += conf/janus.plugin.videocall.cfg.sample
endif

if ENABLE_PLUGIN_VIDEOROOM
plugin_LTLIBRARIES += plugins/libjanus_videoroom.la
plugins_libjanus_videoroom_la_SOURCES = plugins/janus_videoroom.c
plugins_libjanus_videoroom_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_videoroom_la_LDFLAGS = $(plugins_ldflags)
plugins_libjanus_videoroom_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.videoroom.cfg.sample
EXTRA_DIST += conf/janus.plugin.videoroom.cfg.sample
endif

if ENABLE_PLUGIN_VOICEMAIL
plugin_LTLIBRARIES += plugins/libjanus_voicemail.la
plugins_libjanus_voicemail_la_SOURCES = plugins/janus_voicemail.c
plugins_libjanus_voicemail_la_CFLAGS = $(plugins_cflags)
plugins_libjanus_voicemail_la_LDFLAGS = $(plugins_ldflags) -logg
plugins_libjanus_voicemail_la_LIBADD = $(plugins_libadd)
conf_DATA += conf/janus.plugin.voicemail.cfg.sample
EXTRA_DIST += conf/janus.plugin.voicemail.cfg.sample
endif

##
# Post-processing
##

if ENABLE_POST_PROCESSING
bin_PROGRAMS += janus-pp-rec

janus_pp_rec_SOURCES = \
	postprocessing/pp-opus.c \
	postprocessing/pp-opus.h \
	postprocessing/pp-rtp.h \
	postprocessing/pp-webm.c \
	postprocessing/pp-webm.h \
	postprocessing/janus-pp-rec.c \
	$(NULL)

janus_pp_rec_CFLAGS = \
	$(AM_CFLAGS) \
	$(POST_PROCESSING_CFLAGS) \
	$(NULL)

janus_pp_rec_LDADD = \
	$(POST_PROCESSING_LIBS) \
	$(NULL)
endif

##
# Docs
##

if ENABLE_DOCS
SUBDIRS += docs
endif

##
# Configuration
##

configs:
	$(MKDIR_P) $(DESTDIR)$(confdir)
	$(foreach config,$(conf_DATA),cp "$(CURDIR)/$(config)" "$(DESTDIR)$(confdir)/$(notdir $(basename $(config) .sample))";)

include $(top_srcdir)/Makefile.common

AM_CFLAGS += $(WARN_CFLAGS)
AM_CPPFLAGS += $(WARN_CPPFLAGS)
AM_CXXFLAGS += $(WARN_CXXFLAGS)

INCLUDES = -I$(top_srcdir)/src -isystem $(top_srcdir)/contrib

noinst_LIBRARIES = libposix.a
libposix_a_SOURCES = \
	FileSystemPosix.cpp \
	OSPosix.cpp

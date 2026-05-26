bin_PROGRAMS = miltertest

miltertest_SOURCES = miltertest.c
miltertest_CPPFLAGS = $(LIBMILTER_INCDIRS) $(LUA_CFLAGS)
miltertest_CFLAGS = $(PTHREAD_CFLAGS)
miltertest_LDFLAGS = $(LIBMILTER_LIBDIRS) $(PTHREAD_CFLAGS)
miltertest_LDADD = $(LIBMILTER_LIBS) $(LUA_LIBS) $(LIBNSL_LIBS)

man_MANS = miltertest.8

EXTRA_DIST = miltertest.8

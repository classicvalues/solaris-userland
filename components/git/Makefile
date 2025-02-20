#
# CDDL HEADER START
#
# The contents of this file are subject to the terms of the
# Common Development and Distribution License (the "License").
# You may not use this file except in compliance with the License.
#
# You can obtain a copy of the license at usr/src/OPENSOLARIS.LICENSE
# or http://www.opensolaris.org/os/licensing.
# See the License for the specific language governing permissions
# and limitations under the License.
#
# When distributing Covered Code, include this CDDL HEADER in each
# file and include the License file at usr/src/OPENSOLARIS.LICENSE.
# If applicable, add the following below this CDDL HEADER, with the
# fields enclosed by brackets "[]" replaced with your own identifying
# information: Portions Copyright [yyyy] [name of copyright owner]
#
# CDDL HEADER END
#

#
# Copyright (c) 2011, 2022, Oracle and/or its affiliates.
#

BUILD_BITS= 64
# Studio compiler failed to build ref-filter.c:
# "ref-filter.c", line 223: void function cannot return value
COMPILER= gcc
include ../../make-rules/shared-macros.mk

COMPONENT_NAME=		git
COMPONENT_VERSION=	2.35.1
COMPONENT_PROJECT_URL=	http://git-scm.com/
COMPONENT_ARCHIVE=	$(COMPONENT_SRC).tar.xz
COMPONENT_ARCHIVE_HASH=	\
    sha256:d768528e6443f65a203036266f1ca50f9d127ba89751e32ead37117ed9191080
COMPONENT_ARCHIVE_URL=	https://www.kernel.org/pub/software/scm/git/$(COMPONENT_ARCHIVE)
COMPONENT_ANITYA_ID=	5350

COMPONENT_BAID=		116118

#
# man pages are a separate archive
#
COMPONENT_SRC_1 =	$(COMPONENT_NAME)-manpages-$(COMPONENT_VERSION)
COMPONENT_ARCHIVE_1 =	$(COMPONENT_SRC_1).tar.xz
COMPONENT_ARCHIVE_HASH_1 = \
    sha256:e4e3a751f2c05959222c3ea2f0f09481700eca8f915d1398bb270eb6846c3803
COMPONENT_ARCHIVE_URL_1 = https://www.kernel.org/pub/software/scm/git/$(COMPONENT_ARCHIVE_1)
UNPACK_ARGS_1 =		-r $(COMPONENT_SRC_1)
PKG_PROTO_DIRS +=	$(COMPONENT_SRC_1)

ifeq ($(MACH),i386)
# Disable the unit tests since the test results are outdated.
$(SKIP_TEST_AT_TOP_LEVEL)
endif

include $(WS_MAKE_RULES)/common.mk

PYTHON_VERSION = 3.7

# Deliver hardlinks as they appear in the proto area
PKG_AUTO_HARDLINKS=1

PKG_MACROS +=		PYVER=$(PYTHON_VERSION)

CONFIGURE_LIBDIR.64 =	$(CONFIGURE_PREFIX)/lib
CONFIGURE_SCRIPT    =	$(@D)/configure
CPPFLAGS += "-I/usr/include/pcre"

# Needed to use the SHA1 implementation from OpenSSL (rather than git's).
# All three are needed otherwise the git top-level Makefile outputs:
# * new build flags
# and rebuilds everything using the git SHA1 implementation from sha1dc/sha1.c
COMPONENT_BUILD_ENV +=	OPENSSL_SHA1=1
COMPONENT_INSTALL_ENV += OPENSSL_SHA1=1
COMPONENT_TEST_ENV +=	OPENSSL_SHA1=1

CONFIGURE_OPTIONS  +=	--sysconfdir=$(ETCDIR)
CONFIGURE_OPTIONS  +=	--libexecdir=$(USRLIBDIR)
CONFIGURE_OPTIONS  +=	--with-libpcre2
CONFIGURE_OPTIONS  +=	--with-perl=$(PERL)
CONFIGURE_OPTIONS  +=	--with-python=$(PYTHON)

# Build fails if environment variable PROFILE is set.
COMPONENT_BUILD_ENV+=	PROFILE=
COMPONENT_INSTALL_ENV+=	PROFILE=
COMPONENT_TEST_ENV+=	PROFILE=

# Always show the build and link lines for easier debugging.
COMPONENT_BUILD_ARGS+=	V=1

PERL_CONFIG = $(shell $(PERL) -MConfig -e 'print "$$Config{$(1)}"')

# We want the perl modules to install the vendor_perl directory, and their
# manpages to install in the normal perl manpage directory, but the default
# behavior is to put the modules in /usr/lib.  Setting these two variables
# massages the perl to get files to go where we want.
COMPONENT_INSTALL_ARGS+=	INSTALL="$(INSTALL)"
COMPONENT_INSTALL_ARGS+=  perllibdir=/usr/perl5/vendor_perl/$(PERL_VERSION)

# get rid of terminal escape codes in test output
COMPONENT_TEST_ENV += TERM=dumb

# Some tests (like t5539-fetch-http-shallow.sh) fail if the proxies are set.
COMPONENT_TEST_ENV += http_proxy=
COMPONENT_TEST_ENV += https_proxy=
COMPONENT_TEST_ENV += HTTP_PROXY=
COMPONENT_TEST_ENV += HTTPS_PROXY=

# The test results differ for x86 and SPARC
COMPONENT_TEST_MASTER = $(COMPONENT_TEST_RESULTS_DIR)/results-$(MACH$(BITS)).master

# Some tests (like t9402-git-cvsserver-refs.sh) need GNU diff.
COMPONENT_TEST_ENV += PATH="$(GNUBIN):$(PATH)"

# many failures; keep going; later versions of git are much cleaner
COMPONENT_TEST_ARGS += -k -i
# If GITTEST_JOBS is set, then run the jobs in parallel (to that number)
COMPONENT_TEST_ARGS += $(if $(GITTEST_JOBS),-j$(GITTEST_JOBS),)
COMPONENT_TEST_TARGETS = test
# Enable tests requiring Apache
COMPONENT_TEST_TARGETS += LIB_HTTPD_PATH=/usr/apache2/2.4/bin/httpd
COMPONENT_TEST_TARGETS += LIB_HTTPD_MODULE_PATH=/usr/apache2/2.4/libexec
# Put each test result in a file; necessary if jobs run in parallel
COMPONENT_TEST_TARGETS += GIT_TEST_OPTS=--tee
COMPONENT_TEST_TARGETS += DEFAULT_TEST_TARGET=test-noclean
# Compile the test results and put that into the test output file instead
COMPONENT_POST_TEST_ACTION = \
	(cd $(COMPONENT_TEST_DIR)/t/test-results; for i in *.out; do \
		echo "*** $${i%.out}.sh ***"; \
		cat $$i; \
	done; \
	cd ..; \
	$(GMAKE) -s aggregate-results) &> $(COMPONENT_TEST_OUTPUT)

# Some of the tests do not run always in the same order, so the test report
# may be different each time.  To avoid this we will use only the final stats
# as the actual test results.
COMPONENT_TEST_TRANSFORMS += \
	'-n ' \
	'-e "/^fixed/p" '\
	'-e "/^success/p" '\
	'-e "/^failed/p" '\
	'-e "/^broken/p" '\
	'-e "/^total/p" '

# This runs configure but configure doesn't generate a Makefile.
# Instead a Makefile comes with git.
# Anyways viewpathing doesn't work.
# Therefore we need cloney to copy a set of files to build.

COMPONENT_PRE_CONFIGURE_ACTION = \
	($(CLONEY) $(SOURCE_DIR) $(@D))
COMPONENT_PRE_CONFIGURE_ACTION += ; (cd $(@D); $(GMAKE) configure)

# Allow manigest-generate see man pages from the second archive
GENERATE_PROTO_DIRS += $(COMPONENT_SRC_1)

REQUIRED_PACKAGES += crypto/gnupg
REQUIRED_PACKAGES += library/expat
REQUIRED_PACKAGES += library/pcre2
REQUIRED_PACKAGES += library/security/openssl
REQUIRED_PACKAGES += library/zlib
REQUIRED_PACKAGES += $(PERL_PKG)
REQUIRED_PACKAGES += runtime/python-37
REQUIRED_PACKAGES += shell/bash
REQUIRED_PACKAGES += shell/ksh93
REQUIRED_PACKAGES += web/curl
REQUIRED_PACKAGES += web/server/apache-24

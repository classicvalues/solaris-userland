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
# Copyright (c) 2011, 2021, Oracle and/or its affiliates.
#

include ../../../make-rules/shared-macros.mk
COMPILER=	gcc

COMPONENT_NAME=		pycurl
COMPONENT_VERSION=	7.44.1
COMPONENT_PROJECT_URL=	http://pycurl.sourceforge.net/
COMPONENT_SRC=		$(COMPONENT_NAME)-$(COMPONENT_VERSION)
COMPONENT_ARCHIVE=	$(COMPONENT_SRC).tar.gz
COMPONENT_ARCHIVE_HASH=	\
    sha256:5f3a334aca25380fd62ebd46ffaef95aab7ed3bafeda78723d3833c5966ccf04
COMPONENT_ARCHIVE_URL=	$(call pypi_url)
COMPONENT_BUGDB=	python-mod/pycurl
COMPONENT_ANITYA_ID=	7973

COMPONENT_BAID=		113403

PYTHON_VERSIONS = $(PYTHON3_VERSIONS)

include $(WS_MAKE_RULES)/prep.mk
include $(WS_MAKE_RULES)/setup.py.mk
include $(WS_MAKE_RULES)/ips.mk

# to make sure that the docs get installed in the right place.
PYTHON_DATA = /usr

# make sure that correct SSL backend is used (see 32568507)
COMPONENT_SETUP_ARGS += --with-openssl

# Testing environment must be clean
test: ENV += -i

# Rebuild downloaded .so files for each Python version
COMPONENT_PRE_TEST_ACTION = ( \
	cd $(COMPONENT_SRC)/tests/fake-curl/libcurl ; \
	$(RM) *.so ; $(ENV) $(COMPONENT_BUILD_ENV) $(GMAKE) )

COMPONENT_TEST_DIR =	$(SOURCE_DIR)/tests
COMPONENT_TEST_CMD =	$(PYTHON) -m pytest
COMPONENT_TEST_ARGS =

ASLR_MODE = $(ASLR_NOT_APPLICABLE)

# common targets
build:		$(BUILD_32_and_64)

install:	$(INSTALL_32_and_64)

# test requires bottle and flaky which haven't been integrated yet.
test:		$(NO_TESTS)

system-test:    $(SYSTEM_TESTS_NOT_IMPLEMENTED)


REQUIRED_PACKAGES += library/security/openssl
REQUIRED_PACKAGES += system/library
REQUIRED_PACKAGES += web/curl

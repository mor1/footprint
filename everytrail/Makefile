# Copyright (C) 2014 Richard Mortier <mort@cantab.net>.  All Rights
# Reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307,
# USA.

help:
	@egrep "^.*: " Makefile | grep -v "@egrep"

include Makefile.secrets

PPRINT = xmllint --format -

USERID = $(shell \
	$(CURL) "$(ROOT)/user/login?$(USER)" | xmllint --xpath "//userID/text()" -)

USER = username=$(USERNAME)&password=$(PASSWORD)
CURL = curl -k -u $(APIKEY):$(APISECRET)
ROOT = https://www.everytrail.com/api

all: trips pictures

trips.xml: # fetch XML of all trips
	$(CURL) "$(ROOT)/user/trips?$(USER)&user_id=$(USERID)" | $(PPRINT) >| $@

trip-%/data.xml: # fetch XML for a specific trip
	mkdir $$(dirname $@)
	$(CURL) "$(ROOT)/trip/data?$(USER)&trip_id=$*" | $(PPRINT) >| $@
	make pictures-$*

trips: trips.xml # fetch XML for every trip
	for i in $$(xmllint --xpath "//trip/@id" trips.xml) ; do \
		make trip-$${i:4:7}/data.xml ; \
	done

pictures-%: # fetch fullsize pictures for a specific trip
	for p in $$(xmllint --format --xpath '//fullsize' trip-$*/data.xml | \
		gawk -F'</?fullsize>' '{ for (i=2; i <= NF; i+=2) print $$i }' \
	); do \
	  curl --create-dirs -o trip-$*/$$(basename $$p) $$p ; \
	done

pictures: trips.xml # fetch fullsize pictures for all trips
	for i in $$(xmllint --xpath "//trip/@id" trips.xml) ; do \
		make pictures-$${i:4:7} ; \
	done

clean: # remove outputs
	$(RM) *.xml
	$(RM) -r trip-*

## some other unused targets

track-%.xml:
	$(CURL) "$(ROOT)/trip/tracks?$(USER)&trip_id=$*" | $(PPRINT) >| $@

tracks: trips.xml
	for i in $$(xmllint --xpath "//trip/@id" trips.xml) ; do \
		make track-$${i:4:7}.xml ; \
	done

waypoints-%.xml:
	$(CURL) "$(ROOT)/trip/waypoints?$(USER)&trip_id=$*" | $(PPRINT) >| $@

waypoints: trips.xml
	for i in $$(xmllint --xpath "//trip/@id" trips.xml) ; do \
		make waypoints-$${i:4:7}.xml ; \
	done

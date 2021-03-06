# Copyright (c) 2013-2016 The Khronos Group Inc.
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and/or associated documentation files (the
# "Materials"), to deal in the Materials without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Materials, and to
# permit persons to whom the Materials are furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Materials.
#
# THE MATERIALS ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
# CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
# TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
# MATERIALS OR THE USE OR OTHER DEALINGS IN THE MATERIALS.

# Generator scripts and options
# GENOPTS can be e.g. '-noprotect'

PYTHON ?= python3
PYFILES = genheaders.py reg.py
GENOPTS =
GENHEADERS = genheaders.py $(GENOPTS)

# Generate all outputs for Vulkan, including headers and (soon) Asciidoc
# frameworks. Targets:
#
# default - regenerate ../vulkan/vulkan.h
# install - install regenerated vulkan.h
# full_install - install regenerated vulkan.h and ref page API include
#   files. Normally only Jon should do this when accepting a merge into
#   master.
# validate - run XML validator on vk.xml against the schema
# test - check if vulkan.h compiles
# readme.pdf - regnerate schema / HOWTO documentation
# vulkan.h - just regenerate vulkan.h locally (for compile testing)

HEADERS     = ../vulkan/vulkan.h
DOCINCLUDES = vulkan-docs
DOCROOT     = ../../doc/specs/vulkan
DOCPYSRC    = $(DOCROOT)/vkapi.py
DOCVALIDITY = validity
DOCHOSTSYNCTABLE = hostsynctable
XMLDOC	    = readme.pdf

# Could add $(XMLDOC) to default, but that requires a LaTeX install.
# Could regenerate vk.json automatically but the generator script isn't
# creating semantically valid JSON yet, and it's autogenerated, so
# don't.

default install: $(HEADERS)

full_install: default $(DOCINCLUDES) $(DOCPYSRC) $(DOCVALIDITY) $(DOCHOSTSYNCTABLE)

################################################

# Python and XML files on which vulkan.h depends
VKH_DEPENDS = vk.xml genvk.py reg.py generator.py

# Autogenerate installed header from XML API description
../vulkan/vulkan.h: $(VKH_DEPENDS)
	$(PYTHON) genvk.py $(GENOPTS) $@

# Autogenerate API Asciidoc include files from XML
# The target is generated by genvk.py, but is just
# an empty file used as a proxy for when the many
# files in the docs tree were regenerated.
$(DOCINCLUDES): $(VKH_DEPENDS)
	$(PYTHON) genvk.py $(GENOPTS) $@

# Generate API Python dictionaries from XML.
# These are used in validating spec / man pages.
$(DOCPYSRC): $(VKH_DEPENDS)
	$(PYTHON) genvk.py $(GENOPTS) $@

# Generate API Asciidoc validity include files from XML.
$(DOCVALIDITY): $(VKH_DEPENDS)
	$(PYTHON) genvk.py $(GENOPTS) $@

# Generate API Asciidoc hostsynctable include files from XML.
$(DOCHOSTSYNCTABLE): $(VKH_DEPENDS)
	$(PYTHON) genvk.py $(GENOPTS) $@

# Verify registry XML file against the schema
validate:
	jing -c registry.rnc vk.xml

# Test that vulkan.h compiles
test: ../vulkan/vulkan.h ../vulkan/vk_platform.h
	gcc -Wall -pedantic -std=c99 -c -I.. test.c
	g++ -Wall -c -std=c++98 -I.. test.c
	g++ -Wall -c -std=c++11 -I.. test.c
	rm test.o

# Convert XML API description to JSON (not used yet)
vk.json: tojson.py vk.xml
	$(PYTHON) tojson.py vk.xml > $@

################################################

# Documentation targets
readme.pdf: readme.tex Makefile
	touch readme.ind
	pdflatex readme.tex
	pdflatex readme.tex
	makeindex readme.idx
	pdflatex readme.tex

################################################

# Administrative stuff

# Add entries from function_properties.xls (encoded into a Python
# dictionary) to vk.xml

addProps:
	$(PYTHON) rewrite.py vk.xml | $(PYTHON) realign.py > new.xml

################################################

# Files to clean up
TEXDIRT = readme.aux readme.idx readme.ilg readme.ind readme.log readme.toc readme.out
PYDIRT = diag.txt dumpReg.txt errwarn.txt *.pyc regdump.txt
DIRT = $(TEXDIRT) $(PYDIRT) ERRS \#*

clean:
	-rm -f $(DIRT)

# Remove not just intermediate and junk files, but generated targets
# (installed header & asciidoc includes)
clobber: clean
	-rm -f $(HEADERS) $(DOCINCLUDES)
	-rm -f $(DOCROOT)/vkapi.py
	-rm -f $(DOCROOT)/structs/*.txt $(DOCROOT)/protos/*.txt
	-rm -f $(DOCROOT)/enums/*.txt $(DOCROOT)/flags/*.txt
	-rm -f $(DOCVALIDITY)
	-rm -f $(DOCROOT)/validity/protos/*.txt
	-rm -f $(DOCROOT)/validity/structs/*.txt
	-rm -f $(DOCHOSTSYNCTABLE)
	-rm -f $(DOCROOT)/hostsynctable/*.txt

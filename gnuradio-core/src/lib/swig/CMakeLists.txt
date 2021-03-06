# Copyright 2010-2011 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

########################################################################
include(GrPython)
include(GrSwig)

set(GR_SWIG_INCLUDE_DIRS
    ${CMAKE_CURRENT_BINARY_DIR}
    ${GNURADIO_CORE_SWIG_INCLUDE_DIRS}
    ${GRUEL_INCLUDE_DIRS}
    ${LOG4CPP_INCLUDE_DIRS}
    ${Boost_INCLUDE_DIRS}
)
set(GR_SWIG_LIBRARIES
    gnuradio-core
    ${LOG4CPP_LIBRARIES}
)

# Only use if log4cpp is installed
# Define ENABLE_GR_LOG for the .i file to ignore it.
if(ENABLE_GR_LOG)
     SET(GR_SWIG_FLAGS "-DENABLE_GR_LOG")
endif(ENABLE_GR_LOG)

link_directories(${Boost_LIBRARY_DIRS})

########################################################################
# Build and install the swig targets
########################################################################
# ----------------------------------------------------------------
# We've split the previously monstrous gnuradio_corethon into 6
# smaller pieces.  This reduces compile time coupling and creates
# smaller pieces for the compiler to digest.  prior to this change, on
# X86_64, g++'s resident set size was 650MB!
# ----------------------------------------------------------------

set(GR_SWIG_TARGET_DEPS general_generated gengen_generated filter_generated pmt_swig)

foreach(what runtime general gengen filter io hier)
    SET(GR_SWIG_DOC_FILE ${CMAKE_CURRENT_BINARY_DIR}/${what}_swig_doc.i)
    SET(GR_SWIG_DOC_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../${what} ${CMAKE_CURRENT_BINARY_DIR}/../${what})
    GR_SWIG_MAKE(gnuradio_core_${what} gnuradio_core_${what}.i)
    GR_SWIG_INSTALL(
        TARGETS gnuradio_core_${what}
        DESTINATION ${GR_PYTHON_DIR}/gnuradio/gr
        COMPONENT "core_python"
    )
    install(
        FILES
        gnuradio_core_${what}.i
        ${CMAKE_CURRENT_BINARY_DIR}/${what}_swig_doc.i
        DESTINATION ${GR_INCLUDE_DIR}/gnuradio/swig
        COMPONENT "core_swig"
    )
    list(APPEND core_swig_deps ${SWIG_MODULE_gnuradio_core_${what}_REAL_NAME})
endforeach(what)

add_custom_target(core_swig DEPENDS ${core_swig_deps})

########################################################################
# Install various files
########################################################################
install(FILES
    gnuradio.i
    gr_swig_block_magic.i
    gr_shared_ptr.i
    DESTINATION ${GR_INCLUDE_DIR}/gnuradio/swig
    COMPONENT "core_swig"
)

GR_PYTHON_INSTALL(
    FILES gnuradio_core.py
    DESTINATION ${GR_PYTHON_DIR}/gnuradio/gr
    COMPONENT "core_python"
)

install(
    FILES ${CMAKE_CURRENT_SOURCE_DIR}/gnuradio_swig_bug_workaround.h
    DESTINATION ${GR_INCLUDE_DIR}/gnuradio
    COMPONENT "core_swig"
)

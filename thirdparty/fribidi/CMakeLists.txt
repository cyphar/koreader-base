project(fribidi)
cmake_minimum_required(VERSION 3.5.1)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/../cmake_modules")
include("koreader_thirdparty_common")
include("koreader_thirdparty_git")

enable_language(C)

assert_var_defined(CC)
assert_var_defined(CFLAGS)
assert_var_defined(CHOST)
assert_var_defined(LDFLAGS)

ep_get_source_dir(SOURCE_DIR)
ep_get_binary_dir(BINARY_DIR)

set(CFG_ENV_VAR "CC=\"${CC}\" CFLAGS=\"${CFLAGS}\" LDFLAGS=\"${LDFLAGS}\"")
set(CFG_OPTS "-q --prefix=${BINARY_DIR} --disable-static --enable-shared --host=\"${CHOST}\"")
set(CFG_CMD sh -c "${CFG_ENV_VAR} ${SOURCE_DIR}/configure ${CFG_OPTS}")

# Disable building docs, as it requires c2man
set(CFG_CMD "${CFG_CMD} && ${ISED} \"s|SUBDIRS = gen.tab lib bin doc test|SUBDIRS = gen.tab lib bin test|g\" Makefile")

if($ENV{ANDROID})
    set(CFG_CMD "${CFG_CMD} && ${ISED} 's|version_type=none|version_type=linux|' libtool")
    set(CFG_CMD "${CFG_CMD} && ${ISED} 's|need_lib_prefix=no|need_lib_prefix=yes|' libtool")
    set(CFG_CMD "${CFG_CMD} && ${ISED} 's|need_version=no|need_version=yes|' libtool")
    set(CFG_CMD "${CFG_CMD} && ${ISED} 's|library_names_spec=.*|library_names_spec=\"\\\\\$libname\\\\\$release\\\\\$shared_ext\\\\\$versuffix \\\\\$libname\\\\\$release\\\\\$shared_ext\\\\\$major \\\\\$libname\\\\\$shared_ext\"|' libtool")
    set(CFG_CMD "${CFG_CMD} && ${ISED} 's|soname_spec=.*|soname_spec=\"\\\\\$libname\\\\\$release\\\\\$shared_ext\\\\\$major\"|' libtool")
endif()

ko_write_gitclone_script(
    GIT_CLONE_SCRIPT_FILENAME
    https://github.com/fribidi/fribidi.git
    tags/v1.0.11
    ${SOURCE_DIR}
)

include(ExternalProject)
ExternalProject_Add(
    ${PROJECT_NAME}
    DOWNLOAD_COMMAND ${CMAKE_COMMAND} -P ${GIT_CLONE_SCRIPT_FILENAME}
    PATCH_COMMAND NOCONFIGURE=1 ./autogen.sh
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND COMMAND ${CFG_CMD}
    BUILD_COMMAND ${KO_MAKE_RECURSIVE} -j${PARALLEL_JOBS}
    INSTALL_COMMAND ${KO_MAKE_RECURSIVE} -j${PARALLEL_JOBS} install
)

######################################################################
# Defines
######################################################################
BUILD_DATE=`date +"%Y%m%d"`

TINDERBOX_HOME=/workspace
TINDERBOX_SNAPSHOTS_DIR=${TINDERBOX_HOME}/snapshots
TINDERBOX_BIN_DIR=${TINDERBOX_SNAPSHOTS_DIR}/v1_10/${BUILD_DATE}

WIN32_MAKERUNTIME=${TINDERBOX_HOME}/make-runtime.sh
WIN32_MINGW_ENV=${TINDERBOX_HOME}/env
WIN32_ISCC=${TINDERBOX_HOME}/inno/app/ISCC.exe

WORKRAVE_GIT_DIR=${TINDERBOX_HOME}/source/workrave
CONF_FLAGS = --quiet

WORKRAVE_GIT_BRANCH=branch_v1_10

WORKRAVE_LONG_GIT_VERSION=`( cd ${WORKRAVE_GIT_DIR} ; git describe --tags --abbrev=10 --dirty 2>/dev/null )`
WORKRAVE_GIT_VERSION=`( cd ${WORKRAVE_GIT_DIR} ; git describe --tags --abbrev=10 --dirty 2>/dev/null | sed -e 's/-g.*//' )`
1WORKRAVE_VERSION=`cat ${WORKRAVE_GIT_DIR}/configure.ac | grep AM_INIT_AUTOMAKE | cut -d ','  -f2 | cut -d' ' -f2 | cut -d')' -f1`
#WORKRAVE_PREVERSION=`cat ${WORKRAVE_GIT_DIR}/configure.ac | grep AM_INIT_AUTOMAKE | perl -ne ' $$_ =~ /.*([0-9]+).([0-9]+).([0-9]+)/ ; printf "%d.%d.%d.99", $$1, $$2, ($$3 - 1) '`

unix2dos=perl -e 'while (<>) { s/$$/\r/; print; }'

######################################################################
# Miscellaneous
######################################################################

usage:
	@echo Usage: "make all" to make all files from scratch

all: setup workrave-win32-all

debug: workrave-win32-debug-setup

setup:
	( cd ${WORKRAVE_GIT_DIR}; \
	  mkdir -p ${TINDERBOX_BIN_DIR}; \
  )

######################################################################
# GIT
######################################################################

git-pull:
	( cd ${WORKRAVE_GIT_DIR}; \
    git clean -fdx; \
    git checkout ${WORKRAVE_GIT_BRANCH}; \
	  git reset --hard HEAD; \
	  git pull; \
	  git describe --tags --abbrev=10 --dirty )

######################################################################
# Windows
######################################################################

workrave-win32-dir=\
	( cd ${1}; \
    git clean -fdx; \
	  cd ${1}/frontend/gtkmm/win32/setup/; \
	  ${WIN32_MAKERUNTIME} ${WORKRAVE_GIT_DIR}; )

workrave-win32-build=\
	( cd ${1}/common/win32/harpoonHelper/src; \
    mkdir Release; \
	  cp -a ${TINDERBOX_HOME}/prebuilt/${3}64/harpoonHelper.exe Release; \
	  cd ${1}/common/win32/harpoon/src; \
    mkdir Release; \
	  cp -a ${TINDERBOX_HOME}/prebuilt/${3}/harpoon.dll Release; \
	  cp -a ${TINDERBOX_HOME}/prebuilt/${3}64/harpoon64.dll Release; \
    cd ${1}/frontend/applets/win32/src; \
    mkdir Release; \
	  cp -a ${TINDERBOX_HOME}/prebuilt/${3}/workrave-applet.dll Release; \
	  cp -a ${TINDERBOX_HOME}/prebuilt/${3}64/workrave-applet64.dll Release); \
	( cd ${1}; \
	  . ${WIN32_MINGW_ENV}; \
	  bash ./build/win32/autogencross.sh ${CONF_FLAGS} ${2} && make ; \
	  cp -a ${1}/frontend/gtkmm/src/.libs/workrave.exe ${1}/frontend/gtkmm/src; \
	  $$TARGET-nm -nosC --line-numbers ${1}/frontend/gtkmm/src/workrave.exe >${1}/frontend/gtkmm/src/workrave.sym)

workrave-win32-strip=\
	(  . ${WIN32_MINGW_ENV}; \
	$$TARGET-strip ${1}/frontend/gtkmm/src/workrave.exe )

workrave-win32-setup=\
	( cd ${1}/frontend/gtkmm/win32/setup/; \
	  unix2dos setup.iss; \
	  wine "${WIN32_ISCC}" setup.iss ); \
	( cd ${WORKRAVE_GIT_DIR}; \
	  git status; ); \
	  mv ${1}/frontend/gtkmm/win32/setup/Output/setup.exe ${TINDERBOX_BIN_DIR}/workrave-win32-${WORKRAVE_LONG_GIT_VERSION}-${BUILD_DATE}-${2}.exe; \
	  bzip2 -c ${1}/frontend/gtkmm/src/workrave.sym >${TINDERBOX_BIN_DIR}/workrave-win32-${WORKRAVE_LONG_GIT_VERSION}-${BUILD_DATE}-${2}.sym.bz2;

workrave-win32-release-dir:
	$(call workrave-win32-dir,${WORKRAVE_GIT_DIR})

workrave-win32-release-build: workrave-win32-release-dir
	$(call workrave-win32-build,${WORKRAVE_GIT_DIR},--disable-debug,Release)
	$(call workrave-win32-strip,${WORKRAVE_GIT_DIR})

workrave-win32-release-setup: workrave-win32-release-build
	$(call workrave-win32-setup,${WORKRAVE_GIT_DIR},installer)

workrave-win32-debug-dir: 
	$(call workrave-win32-dir,${WORKRAVE_GIT_DIR})

workrave-win32-debug-build: workrave-win32-debug-dir
	$(call workrave-win32-build,${WORKRAVE_GIT_DIR},--enable-debug --enable-tracing,Debug)

workrave-win32-debug-setup: workrave-win32-debug-build
	$(call workrave-win32-setup,${WORKRAVE_GIT_DIR},debug-installer)
	( cd ${WORKRAVE_GIT_DIR}/frontend/gtkmm/src; \
	  zip ${TINDERBOX_BIN_DIR}/workrave-win32-${WORKRAVE_LONG_GIT_VERSION}-${BUILD_DATE}-debug.zip workrave.exe )

workrave-win32-all: \
		workrave-win32-release-setup \
		workrave-win32-debug-setup 

######################################################################
# Source
######################################################################

workrave-source: 
	( cd ${WORKRAVE_GIT_DIR}; \
	  ./autogen.sh --disable-gnome \
	  && make dist \
	  && mv ${WORKRAVE_GIT_DIR}/workrave-${WORKRAVE_VERSION}.tar.gz ${TINDERBOX_BIN_DIR}/workrave-src-${WORKRAVE_LONG_GIT_VERSION}-${BUILD_DATE}.tar.gz )

######################################################################
# Upload
######################################################################
upload-purge:
	for i in `find ${TINDERBOX_SNAPSHOTS_DIR}/v1_10 -type d -mtime +5 -print`; \
	do \
		echo Purging old snapshot: $$i; \
		rm -rf $$i; \
	done

upload:
	( cd ${TINDERBOX_SNAPSHOTS_DIR} ; \
    rsync -rv --delete --rsh=ssh . workrave@liberator.krandor.org:/home/workrave/public_html )

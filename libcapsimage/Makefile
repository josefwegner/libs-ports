# Set the libc to be used with the LIBC argument
# possible values newlib, clib2, clib4
# make LIBC=clib4
#

LIB_VERSION=5.1
LIBC?=newlib

PKGNAME=capsimage-${LIB_VERSION}

include ../_common/help.mk
include ../_common/checks.mk

.PHONY: download
download:
	@echo "${CCPINK}${CCBOLD}\n-> Download archive ${PKGNAME}\n ${CCEND}"
	@curl -fsSL "https://www.kryoflux.com/download/spsdeclib_${LIB_VERSION}_source.zip" -o ${PKGNAME}.zip && \
		echo "${CCPINK}${CCBOLD}\n-> Extract archive\n${CCEND}" && \
		mkdir ${PKGNAME}_tmp && \
		unzip ${PKGNAME}.zip -d ./${PKGNAME}_tmp && \
		unzip ./${PKGNAME}_tmp/capsimg_source_linux_macosx.zip -d ./ && \
		mv ./capsimg_source_linux_macosx ./${PKGNAME}

#####################################################################
##@ Build

.PHONY: init
init: download patches ## Download the archive and initialize the build
	@cd ${PKGNAME}/CAPSImg && \
		autoconf

.PHONY: patches
patches:
	@echo "${CCPINK}${CCBOLD}\n-> Apply the patches\n${CCEND}"
	@patch -s -p0 < ./patch-${LIB_VERSION}.diff

.PHONY: build
build: ## Build with specified LIBC=(newlib|clib2|clib4)
	@cd ${PKGNAME}/CAPSImg && \
	CC="ppc-amigaos-gcc" \
	CFLAGS="-mcrt=${LIBC}" \
	LDFLAGS="-mcrt=${LIBC}" \
	PKG_CONFIG_PATH=${SDK_PATH}/local/${LIBC}/lib/pkgconfig \
	LIBS="" \
	./configure --host=ppc-amigaos \
		--prefix=${SDK_PATH}/local/${LIBC} \
		--includedir=${SDK_PATH}/local/common/include \
	&& \
	make -j$(shell nproc)

.PHONY: build/%
build/%:
	@echo "${CCPINK}${CCBOLD}\n-> Compiling for $*\n${CCEND}"
	@make build LIBC=$*

#####################################################################
##@ Clean

.PHONY: clean
clean: ## Clean the latest build
	@echo "${CCPINK}${CCBOLD}\n-> Clean the build folder\n${CCEND}"
	@make -C ./${PKGNAME}/CAPSImg clean

.PHONY: cleanall
cleanall: ## Clean the downloaded files and folders
	@rm -rf ./${PKGNAME}.zip ./${PKGNAME} ./${PKGNAME}_tmp ./SDK

#####################################################################
##@ Release

.PHONY: release
release: release-init build/newlib pack/newlib clean build/clib2 pack/clib2 clean build/clib4 pack/clib4 clean archive ## Build and create the release

.PHONY: release-init
release-init: ## Prepare the release folders
	@echo "${CCPINK}${CCBOLD}\n-> Clear previous release folders...\n${CCEND}"
	@rm -rf ./SDK

	@echo "${CCPINK}${CCBOLD}\n-> Create release folders...\n${CCEND}"
	@mkdir -p ./SDK/local/Documentation/capsimage
	@cp ${PKGNAME}_tmp/RELEASE* \
		${PKGNAME}_tmp/LICENCE* \
		${PKGNAME}_tmp/HISTORY* \
		./SDK/local/Documentation/capsimage/

.PHONY: pack/%
pack/%:
	@echo "${CCPINK}${CCBOLD}\n-> Packaging for $*\n${CCEND}"
	@mkdir -p ./SDK/local/$*/include/caps
	@mkdir -p ./SDK/local/$*/lib
	@cp ${PKGNAME}/CAPSImg/libcapsimage.a \
		${PKGNAME}/CAPSImg/libcapsimage.so.${LIB_VERSION} \
		./SDK/local/$*/lib/
	@cp ${PKGNAME}/LibIPF/CapsAPI.h \
		${PKGNAME}/LibIPF/CapsFDC.h \
		${PKGNAME}/LibIPF/CapsForm.h \
		./SDK/local/$*/include/caps/

.PHONY: archive
archive: ## Create the release archive
	@echo "${CCPINK}${CCBOLD}\n-> Creating the lha release file...\n${CCEND}"
	@rm -f ./${PKGNAME}.lha
	@lha aq ./${PKGNAME}.lha ./SDK/

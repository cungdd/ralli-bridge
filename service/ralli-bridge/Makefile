# Copyright (c) 2023-2024 Project CHIP Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

include $(TOPDIR)/rules.mk

PKG_NAME:=ralli-bridge
PKG_RELEASE:=1
PKG_SOURCE_URL:=git@gitlab.com:r-ng-ng/matter/connectedhomeip.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_MIRROR:=0 # don't try OpenWrt mirror
PKG_SOURCE_SUBMODULES:=\
  third_party/pigweed/repo \
  third_party/jsoncpp/repo \
  third_party/nlassert/repo \
  third_party/nlio/repo

# Hash can be regenerated with make package/ralli-bridge/check FIXUP=1

# PKG_SOURCE_DATE:=2025-03-14
# PKG_SOURCE_VERSION:=2d72b7f8daaffcf8a1c106084d858e51d552c41b
# PKG_MIRROR_HASH:=dab0b318da56c4b8f9e503131de15f0ed29c751c4abd474a1c70a3abaaafd712

PKG_SOURCE_DATE:=2025-06-14
PKG_SOURCE_VERSION:=125188b74aadbe6b7c693ac14d259438f1b68f37
PKG_MIRROR_HASH:=dab0b318da56c4b8f9e503131de15f0ed29c751c4abd474a1c70a3abaaafd712

# Use local source dir for development
# USE_SOURCE_DIR:=$(HOME)/workspace/connectedhomeip

PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_PARALLEL:=1
PKG_BUILD_DEPENDS:=gn/host

include $(INCLUDE_DIR)/package.mk

# Hard-code uid/gid due to https://github.com/openwrt/openwrt/issues/13927
define Package/ralli-bridge
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Matter Network Infrastructure Manager Daemon
  URL:=https://github.com/project-chip/connectedhomeip
  DEPENDS:=+libstdcpp +libatomic +libopenssl +jsoncpp +libmosquittopp
  USERID:=matter=5540:matter=5540
endef

define Package/ralli-bridge/description
  Matter Network Infrastructure Manager Daemon
  Integrates a router / access point with the Matter IoT ecosystem.
endef

# https://github.com/project-chip/matter-openwrt/issues/2
# PKG_SOURCE_SUBMODULES support is in OpenWrt main but not 23.x releases.
ifeq ($(DUMP)$(filter SUBMODULES:=,$(Download/Defaults)),)
$(error PKG_SOURCE_SUBMODULES is not supported, ensure https://github.com/openwrt/openwrt/pull/13000 is included in your OpenWrt buildroot)
endif

# The build environment contains host tools that can be shared between targets
CHIP_BUILD_ENV_DIR:=$(STAGING_DIR_HOST)/share/chip-build-env
OUT_DIR:=$(PKG_BUILD_DIR)/out/openwrt

define Build/Configure
	mkdir -p $(OUT_DIR) && cd $(OUT_DIR) && \
		$(CONFIGURE_VARS) $(PKG_BUILD_DIR)/scripts/configure \
			--build-env-dir="$(CHIP_BUILD_ENV_DIR)" \
			--project=examples/bridge-app/linux \
			--target=$(GNU_TARGET_NAME) \
			--logging-backend=syslog \
			--enable-access-restrictions=no \
			--enable-wifi=no \
			--enable-openthread=no \
			--enable-network-layer-ble=no \
			--enable-tracing-support=no \
			--enable-transport-trace=no \
			--use-data-model-interface=disabled
endef

# https://github.com/openwrt/openwrt/issues/13016
TARGET_CFLAGS += -Wno-format-nonliteral

TARGET_CFLAGS += \
	-DCHIP_SYSLOG_IDENT=\"matter\"
	-DCHIP_CONFIG_KVS_PATH=\"/etc/matter/chip_kvs.ini\" \
	-DFATCONFDIR=\"/etc/matter\" \
	-DSYSCONFDIR=\"/etc/matter\" \
	-DLOCALSTATEDIR=\"/etc/matter\"

define Build/Compile
	$(NINJA) -C $(OUT_DIR) chip-bridge-app
endef

define Package/ralli-bridge/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/etc/init.d
	$(INSTALL_BIN) $(OUT_DIR)/chip-bridge-app $(1)/usr/sbin
	$(INSTALL_BIN) ./files/matter.init $(1)/etc/init.d/matter
endef

$(eval $(call BuildPackage,ralli-bridge))

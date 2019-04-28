# port-trigger
kernel module to match the port-ranges, trigger related port-ranges, and alters the destination to a local IP address

# Makefile For Openwrt
```Makefile

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=port-trigger
PKG_VERSION:=1.0
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/dnetlab/port-trigger.git
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=397288350b17baba8a64cfb2b0882a22a04898cd
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.gz

include $(INCLUDE_DIR)/package.mk

define KernelPackage/ipt-trigger
    SUBMENU  := Netfilter Extensions
    TITLE    := TRIGGER target netfilter module
    DEPENDS  := +kmod-ipt-core +kmod-ipt-nat
    FILES    := $(PKG_BUILD_DIR)/module/ipt_TRIGGER.ko
    AUTOLOAD :=$(call AutoLoad,46,ipt_TRIGGER)
endef

define Package/iptables-mod-trigger
    SECTION  := net
    CATEGORY := Network
    SUBMENU  := Firewall
    TITLE    := TRIGGER target iptables extension
    DEPENDS  := iptables +kmod-ipt-trigger
endef

# define Build/Prepare
	# mkdir -p $(PKG_BUILD_DIR)
	# $(CP) ./src/* $(PKG_BUILD_DIR)/
# endef

define Build/Compile/trigger-kmod
	$(MAKE) -C $(LINUX_DIR) \
		CROSS_COMPILE="$(KERNEL_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		SUBDIRS="$(PKG_BUILD_DIR)/module" \
		KERNELDIR=$(LINUX_DIR) \
		CC="$(TARGET_CC)" \
		EXTRA_CFLAGS="-I$(PKG_BUILD_DIR)/header" \
		modules
endef

define Build/Compile/trigger-lib
    $(MAKE) -C $(PKG_BUILD_DIR)/extension \
        $(TARGET_CONFIGURE_OPTS) \
        CFLAGS="$(TARGET_CFLAGS)" \
        LDFLAGS="$(TARGET_LDFLAGS)" \
		IFLAGS="$(TARGET_CPPFLAGS) -I$(PKG_BUILD_DIR)/header" 
endef	

define Build/Compile
	$(Build/Compile/trigger-kmod)
	$(Build/Compile/trigger-lib)
endef

define Package/iptables-mod-trigger/install
	$(INSTALL_DIR) $(1)/usr/lib/iptables
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/extension/*.so $(1)/usr/lib/iptables/
endef

$(eval $(call KernelPackage,ipt-trigger))
$(eval $(call BuildPackage,iptables-mod-trigger))
```

# You may need to add the following two header files to compile port-trigger
# linux-3.14.77/include/net/netfilter/nf_nat_rule.h
```header
#ifndef _NF_NAT_RULE_H
#define _NF_NAT_RULE_H
#include <net/netfilter/nf_conntrack.h>
#include <net/netfilter/nf_nat.h>
#include <linux/netfilter_ipv4/ip_tables.h>

extern int nf_nat_rule_init(void) __init;
extern void nf_nat_rule_cleanup(void);
extern int nf_nat_rule_find(struct sk_buff *skb,
                            unsigned int hooknum,
                            const struct net_device *in,
                            const struct net_device *out,
                            struct nf_conn *ct);

#endif /* _NF_NAT_RULE_H */
```

# linux-3.14.77/include/linux/netfilter_ipv4/ip_autofw.h
```header
/*
 * Copyright (C) 2008, Broadcom Corporation
 * All Rights Reserved.
 *
 * THIS SOFTWARE IS OFFERED "AS IS", AND BROADCOM GRANTS NO WARRANTIES OF ANY
 * KIND, EXPRESS OR IMPLIED, BY STATUTE, COMMUNICATION OR OTHERWISE. BROADCOM
 * SPECIFICALLY DISCLAIMS ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS
 * FOR A SPECIFIC PURPOSE OR NONINFRINGEMENT CONCERNING THIS SOFTWARE.
 *
 * $Id: ip_autofw.h,v 1.1 2008/10/02 03:42:40 Exp $
 */

#ifndef _IP_AUTOFW_H
#define _IP_AUTOFW_H

#define AUTOFW_MASTER_TIMEOUT 600       /* 600 secs */

struct ip_autofw_info {
        u_int16_t proto;        /* Related protocol */
        u_int16_t dport[2];     /* Related destination port range */
        u_int16_t to[2];        /* Port range to map related destination port range to */
};

struct ip_autofw_expect {
        u_int16_t dport[2];     /* Related destination port range */
        u_int16_t to[2];        /* Port range to map related destination port range to */
};

#endif /* _IP_AUTOFW_H */
```

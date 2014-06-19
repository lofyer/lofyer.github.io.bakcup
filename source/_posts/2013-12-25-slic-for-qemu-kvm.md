---
title: Slic for qemu-kvm
author: lofyer
layout: post
comments: true
videourl:
  - 
categories:
  - Virtualization
---
This article is a howto for activation-ready of Windows.  
SLIC 2.0 is for 2003 & XP, and 2.1 for Win7 & 2008  
Original seabios reads slic table from exactly the host. However, if your motherboard(not OEM) happen to own none, you will most probably make one by your self.

### Seabios

You could get the lastest code from here.

<pre>git clone git://git.seabios.org/seabios.git seabios</pre>

Or, you can download from here.

<a href="http://code.coreboot.org/p/seabios/downloads/get/seabios-1.7.2.2.tar.gz" title="seabios-1.7.2.2.tar.gz" target="_blank">seabios-1.7.2.2.tar.gz</a>

### SLIC-BIN

Here&#8217;s a collection of various slic table.  
<a href="http://blog.lofyer.org/slic-for-qemu-kvm/slic-2-1-bins/" rel="attachment wp-att-3483">SLIC 2.1 BINS</a>

### Seaslic patch

This is patch for seabios to make it slic table enabled.  
Download from here.

<a href="https://cloud.lofyer.org/public.php?service=files&#038;t=3aa0db051506d88bce8e6d03d621f47e" title="Seaslic.tar.xz" target="_blank">Seaslic.tar.xz, seabios-1.7.2 compatible</a>  
Here&#8217;s the patch content.

<pre>--- a/src/acpi.c	2013-01-19 06:44:54.000000000 +0600
+++ b/src/acpi.c	2013-05-07 01:16:30.000000000 +0600
@@ -214,6 +214,11 @@

 #include "acpi-dsdt.hex"

+#define CONFIG_OEM_SLIC
+#ifdef CONFIG_OEM_SLIC
+#include "acpi-slic.hex"
+#endif
+
 static void
 build_header(struct acpi_table_header *h, u32 sig, int len, u8 rev)
 {
@@ -226,6 +231,10 @@
     h->oem_revision = cpu_to_le32(1);
     memcpy(h->asl_compiler_id, CONFIG_APPNAME4, 4);
     h->asl_compiler_revision = cpu_to_le32(1);
+    #ifdef CONFIG_OEM_SLIC
+    if (sig == RSDT_SIGNATURE) // only RSDT is checked by win7 &#038; vista
+	memcpy(h->oem_id, ((struct acpi_table_header*)SLIC)->oem_id, 14);
+    #endif
     h->checksum -= checksum(h, len);
 }

@@ -827,6 +836,15 @@
     ACPI_INIT_TABLE(build_srat());
     if (pci->device == PCI_DEVICE_ID_INTEL_ICH9_LPC)
         ACPI_INIT_TABLE(build_mcfg_q35());
+    #ifdef CONFIG_OEM_SLIC
+	void *buf = malloc_high(sizeof(SLIC));
+	if (!buf)
+	    warn_noalloc();
+	else {
+	    memcpy(buf, SLIC, sizeof(SLIC));
+	    ACPI_INIT_TABLE(buf);
+	}
+    #endif

     u16 i, external_tables = qemu_cfg_acpi_additional_tables();</pre>

### Compile

You don&#8217;t have to apply the seaslic patch with patch.sh, you can do that by hand.  
Before you start, do this:

<pre>xxd -i /sys/firmware/acpi/tables/SLIC | grep -v len | sed 's/unsigned ch   ar.*/static char SLIC[] = {/' > seabios.submodule/src/acpi-slic.hex</pre>

Or,

<pre>xxd -i DELL.BIN | grep -v len | sed 's/unsigned ch   ar.*/static char SLIC[] = {/' > seabios.submodule/src/acpi-slic.hex</pre>

After applying the patch, you can compile the bios.bin, and copy that to **/usr/share/qemu-kvm/my-bios.bin** or rewrite **bios.bin** instead.  
Here&#8217;s my bios.bin with Dell[DELL-QA09-NVDA]2.1.BIN from SLIC BIN

<a href="https://cloud.lofyer.org/public.php?service=files&#038;t=996a37a5726b448b81802d641774504f" title="my-bin.tar.xz" target="_blank">my-bin.tar.xz</a>

### Qemu-cmd

<pre>qemu-kvm XXX -bios /usr/share/qemu-kvm/my-bios.bin -acpitable file=Dell[DELL-QA09-NVDA]2.1.BIN</pre>

In the guest, you could see that SLIC by /sys/firmware/acpi/tables/SLIC in Linux or SLIC_Toolkit in Windows.
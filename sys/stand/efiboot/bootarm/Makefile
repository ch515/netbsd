# $NetBSD: Makefile,v 1.1.6.1 2020/01/26 11:21:59 martin Exp $

PROG=		bootarm.efi
OBJFMT=		binary
NEWVERSWHAT=	"efiboot (arm)"
GNUEFIARCH=	arm
LIBGNUEFI_ARCH=	arm

EXTRA_SOURCES=	efibootarm.c
EXTRA_SOURCES+=	cache.S

COPTS+=		-mno-unaligned-access -ffreestanding -fno-unwind-tables
CFLAGS+=	-DEFIBOOT_ALIGN=0x1000000
LDFLAGS+=	-N

.include "${.CURDIR}/../Makefile.efiboot"

release: check_RELEASEDIR
	${HOST_INSTALL_FILE} -m ${BINMODE} ${PROG} \
		${RELEASEDIR}/${MACHINE}/installation/misc

#	$NetBSD: Makefile,v 1.1.2.5 2019/09/01 14:12:22 martin Exp $

NOSANITIZER=    # defined

.include "../common.mk"
.include "../sources.mk"

.include <bsd.own.mk>

TOPDIR=	${NETBSDSRCDIR}/sys/external/bsd/compiler_rt/dist

LIB=	libclang_rt.ubsan_minimal-x86_64

.PATH: ${TOPDIR}/lib/ubsan_minimal

SRCS+=	${UBSAN_MINIMAL_SOURCES}

CXXFLAGS+=	-fno-builtin -fno-exceptions -fno-stack-protector -fno-rtti

LIBDIR=		/usr/lib/clang/${CLANG_VER}/lib/netbsd
LIBISCXX=	yes

_LIB_PREFIX=
MKDEBUGLIB=	no
MKLINT=		no
MKPICINSTALL=	no
MKPROFILE=	no
MKSTATICLIB=	yes

COPTS+=		-fPIC
CPPFLAGS+=	-I${TOPDIR}/lib

.include "../m64.mk"
.include "../syms.mk"

SYMS=		libclang_rt.ubsan_minimal-x86_64.a.syms
SYMS_EXTRA=	${TOPDIR}/lib/ubsan_minimal/ubsan.syms.extra

beforeinstall: ${SYMS}

CLEANFILES+=	${SYMS}
FILES=		${SYMS}
FILESDIR=	${LIBDIR}

.include <bsd.lib.mk>

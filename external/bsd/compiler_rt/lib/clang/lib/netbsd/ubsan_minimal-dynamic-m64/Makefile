#	$NetBSD: Makefile,v 1.1.2.2 2019/08/16 18:44:09 martin Exp $

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

LIBISMODULE=	yes

CPPFLAGS+=	-I${TOPDIR}/lib

.include "../m64.mk"
.include "../shared_deps.mk"

.include <bsd.lib.mk>

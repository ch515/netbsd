#	$NetBSD: Makefile,v 1.2.114.1 2019/12/17 12:45:30 martin Exp $

PROG=	rndctl
MAN=	rndctl.8
SRCS+= rndctl.c

# Hack: libc does not export public SHA-3 symbols, so we'll just copy
# them here statically.
.PATH: ${NETBSDSRCDIR}/common/lib/libc/hash/sha3

# Hack for namespace.h in sha3.c.
CPPFLAGS+=     -I${.CURDIR}

SRCS+= sha3.c
SRCS+= keccak.c

.include <bsd.prog.mk>

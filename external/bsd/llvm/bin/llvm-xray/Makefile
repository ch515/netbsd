#	$NetBSD: Makefile,v 1.3 2018/07/17 18:55:26 joerg Exp $

PROG_CXX=	llvm-xray
NOMAN=		yes

.include <bsd.init.mk>

.PATH: ${LLVM_SRCDIR}/tools/llvm-xray

SRCS=	func-id-helper.cpp \
	llvm-xray.cpp \
	xray-account.cpp \
	xray-color-helper.cpp \
	xray-converter.cpp \
	xray-extract.cpp \
	xray-graph-diff.cpp \
	xray-graph.cpp \
	xray-registry.cpp \
	xray-stacks.cpp

LLVM_LIBS+= \
	XRay \
	DebugInfoSymbolize \
	DebugInfoDWARF \
	DebugInfoPDB \
	DebugInfoPDBNative \
	Object \
	BitReader \
	MCParser \
	IR \
	MC \
	BinaryFormat \
	Support \
	Demangle

LDADD+=	-lz
.if !defined(HOSTPROG)
DPADD+=	${LIBZ}
.endif

.include "${.PARSEDIR}/../../link.mk"

.include <bsd.prog.mk>

#	$OpenBSD: Makefile,v 1.18 2019/05/01 20:53:55 deraadt Exp $

FS=		miniroot${OSrev}.fs
FSSIZE=		9600
FSDISKTYPE=	mini34
CDROM=		cd${OSrev}.iso
MOUNT_POINT=	/mnt
MTREE=		${UTILS}/mtree.conf

LISTS=		${.CURDIR}/../common/list ${.CURDIR}/list.local
UTILS=		${.CURDIR}/../../miniroot

EFIBOOT=	${DESTDIR}/usr/mdec/BOOTX64.EFI ${DESTDIR}/usr/mdec/BOOTIA32.EFI

all: ${FS} ${CDROM}

${FS}: bsd.gz
	dd if=/dev/zero of=${FS} bs=512 count=${FSSIZE}
	vnconfig -v ${FS} > vnd
	fdisk -yi -l ${FSSIZE} -b 960 -f ${DESTDIR}/usr/mdec/mbr `cat vnd`
	disklabel -wAT ${.CURDIR}/template `cat vnd`
	newfs -t msdos /dev/r`cat vnd`i
	mount /dev/`cat vnd`i ${MOUNT_POINT}
	mkdir -p ${MOUNT_POINT}/efi/boot
	cp ${EFIBOOT} ${MOUNT_POINT}/efi/boot
	umount ${MOUNT_POINT}
	newfs -m 0 -o space -i 524288 -c ${FSSIZE} /dev/r`cat vnd`a
	mount /dev/`cat vnd`a ${MOUNT_POINT}
	cp ${DESTDIR}/usr/mdec/boot ${.OBJDIR}/boot
	strip ${.OBJDIR}/boot
	strip -R .comment -R .SUNW_ctf ${.OBJDIR}/boot
	installboot -v -r ${MOUNT_POINT} `cat vnd` \
	    ${DESTDIR}/usr/mdec/biosboot ${.OBJDIR}/boot
	dd if=bsd.gz of=${MOUNT_POINT}/bsd bs=512
	df -i ${MOUNT_POINT}
	umount ${MOUNT_POINT}
	vnconfig -u `cat vnd`
	rm -f vnd

${CDROM}: bsd.rd
	rm -rf ${.OBJDIR}/cd-dir
	mkdir -p ${.OBJDIR}/cd-dir/${OSREV}/amd64
	mkdir -p ${.OBJDIR}/cd-dir/etc
	echo "set image /${OSREV}/amd64/bsd.rd" > ${.OBJDIR}/cd-dir/etc/boot.conf
	cp ${.OBJDIR}/bsd.rd ${.OBJDIR}/cd-dir/${OSREV}/amd64
	cp ${DESTDIR}/usr/mdec/cdbr ${.OBJDIR}/cd-dir/${OSREV}/amd64
	cp ${DESTDIR}/usr/mdec/cdboot ${.OBJDIR}/cd-dir/${OSREV}/amd64/cdboot
	mkhybrid -a -R -T -L -l -d -D -N -o ${.OBJDIR}/${CDROM} \
	    -A "OpenBSD ${OSREV} amd64 bootonly CD" \
	    -P "Copyright (c) `date +%Y` Theo de Raadt, The OpenBSD project" \
	    -p "Theo de Raadt <deraadt@openbsd.org>" \
	    -V "OpenBSD/amd64   ${OSREV} boot-only CD" \
	    -b ${OSREV}/amd64/cdbr -c ${OSREV}/amd64/boot.catalog \
	    ${.OBJDIR}/cd-dir

MRDISKTYPE=	rdrootb
MRMAKEFSARGS=	-o disklabel=${MRDISKTYPE},minfree=0,density=4096

bsd.gz: bsd.rd
	cp bsd.rd bsd.strip
	strip bsd.strip
	strip -R .comment -R .SUNW_ctf bsd.strip
	gzip -c9n bsd.strip > bsd.gz

bsd.rd: mr.fs bsd
	cp bsd bsd.rd
	rdsetroot bsd.rd mr.fs

bsd:
	cd ${.CURDIR}/../../../sys/arch/amd64/compile/RAMDISK_CD && \
	    su ${BUILDUSER} -c '${MAKE} config && ${MAKE} clean && exec ${MAKE}'
	cp -p ${.CURDIR}/../../../sys/arch/amd64/compile/RAMDISK_CD/obj/bsd bsd

mr.fs: instbin
	rm -rf $@.d
	install -d -o root -g wheel $@.d
	mtree -def ${MTREE} -p $@.d -u
	CURDIR=${.CURDIR} OBJDIR=${.OBJDIR} OSrev=${OSrev} \
	    TARGDIR=$@.d UTILS=${UTILS} RELEASEDIR=${RELEASEDIR} \
	    sh ${UTILS}/runlist.sh ${LISTS}
	rm $@.d/instbin
	makefs ${MRMAKEFSARGS} $@ $@.d

instbin.mk instbin.cache instbin.c: instbin.conf
	crunchgen -E -D ${.CURDIR}/../../.. -L ${DESTDIR}/usr/lib \
	    -c instbin.c -e instbin -m instbin.mk instbin.conf

instbin: instbin.mk instbin.cache instbin.c
	${MAKE} -f instbin.mk SRCLIBDIR=${.CURDIR}/../../../lib all
	strip -R .comment -R .SUNW_ctf instbin

instbin.conf: ${LISTS}
	awk -f ${UTILS}/makeconf.awk  ${LISTS} > instbin.conf

unconfig:
	-umount -f ${MOUNT_POINT}
	-[ -f vnd ] && vnconfig -u `cat vnd` && rm -f vnd

.ifdef RELEASEDIR
install:
	cp bsd.rd ${RELEASEDIR}/bsd.rd
	chmod a+r ${RELEASEDIR}/bsd.rd
	cp ${FS} ${RELEASEDIR}
	cp ${CDROM} ${RELEASEDIR}
.endif

clean cleandir:
	rm -f *.core mr.fs instbin instbin.mk instbin*.cache \
	    *.o *.lo *.c bsd bsd.rd bsd.gz bsd.strip floppy*.fs \
	    lib*.a lib*.olist instbin.map boot instbin.conf ${FS}
	rm -rf mr.fs.d
	rm -f ${CDROM}
	rm -rf cd-dir

.include <bsd.obj.mk>

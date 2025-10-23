# .copr/Makefile (version corrigée)

ZFS_TAG ?= zfs-2.3.4
GIT_URL ?= https://github.com/openzfs/zfs.git
SPEC_NAME := $(notdir $(spec))           # <-- récupère juste 'zfs-kmod.spec' ou 'zfs-dkms.spec'
ZFS_SPEC  := zfs/rpm/redhat/$(SPEC_NAME) # <-- spec générée par ./configure dans l'arbre zfs

srpm:
	dnf -y install git autoconf automake libtool make gcc \
	               libtirpc-devel redhat-rpm-config \
	               kernel-srpm-macros || dnf -y install kernel-rpm-macros

	rm -rf zfs
	git clone --depth 1 --branch $(ZFS_TAG) $(GIT_URL) zfs
	cd zfs && ./autogen.sh
	cd zfs && ./configure --with-config=srpm
	cd zfs && make -j$$(nproc) dist

	# Remettre à COPR les sources + la SPEC générée par openzfs
	install -m0644 zfs/zfs-*.tar.* $(outdir)/
	test -f $(ZFS_SPEC) || (echo "Missing $$(pwd)/$(ZFS_SPEC)"; exit 1)
	install -m0644 $(ZFS_SPEC) $(outdir)/

	# Fabriquer le SRPM DANS le chroot (macros kmod dispo ici)
	rpmbuild -bs \
	  --define "_sourcedir $(outdir)" \
	  --define "_srcrpmdir $(outdir)" \
	  $(outdir)/$(SPEC_NAME)

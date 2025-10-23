# .copr/Makefile
ZFS_TAG ?= zfs-2.3.4
GIT_URL ?= https://github.com/openzfs/zfs.git

# Copr passe 'spec' (souvent un chemin ABSOLU) => on prend juste le nom de fichier
SPEC_NAME := $(notdir $(spec))
ZFS_SPEC  := zfs/rpm/redhat/$(SPEC_NAME)

srpm:
	# 1) Outils + macros kmod + RPC headers pour que ./configure et rpmbuild passent
	dnf -y install git autoconf automake libtool make gcc \
	               libtirpc-devel redhat-rpm-config \
	               kernel-srpm-macros || dnf -y install kernel-rpm-macros

	# 2) Récupérer OpenZFS
	rm -rf zfs
	git clone --depth 1 --branch $(ZFS_TAG) $(GIT_URL) zfs

	# 3) Générer les .spec et le tarball
	cd zfs && ./autogen.sh
	cd zfs && ./configure --with-config=srpm
	cd zfs && make -j$$(nproc) dist

	# 4) Fournir à Copr la SPEC générée par OpenZFS + la source
	install -m0644 zfs/zfs-*.tar.* $(outdir)/
	test -f $(ZFS_SPEC) || (echo "Missing $$(pwd)/$(ZFS_SPEC)"; exit 1)
	install -m0644 $(ZFS_SPEC) $(outdir)/

	# 5) Construire le SRPM DANS le chroot (macros kmod dispo ici)
	rpmbuild -bs \
	  --define "_sourcedir $(outdir)" \
	  --define "_srcrpmdir $(outdir)" \
	  $(outdir)/$(SPEC_NAME)

# .copr/Makefile
# Utilisé par Copr (SCM → make srpm). Exécuté DANS un chroot mock.

ZFS_TAG ?= zfs-2.3.4     # ou zfs-2.3-release si tu préfères la branche release
GIT_URL  ?= https://github.com/openzfs/zfs.git

srpm:
	# 1) Outils + macros kmod + headers RPC pour que ./configure passe
	dnf -y install git autoconf automake libtool make gcc \
	               libtirpc-devel \
	               redhat-rpm-config \
	               kernel-srpm-macros || dnf -y install kernel-rpm-macros

	# 2) Récupérer la source OpenZFS
	rm -rf zfs
	git clone --depth 1 --branch $(ZFS_TAG) $(GIT_URL) zfs
	cd zfs && ./autogen.sh

	# 3) Générer les .spec à partir des .spec.in et le tarball
	cd zfs && ./configure --with-config=srpm
	cd zfs && make -j$$(nproc) dist

	# 4) Déposer tarball + spec attendus par Copr dans $(outdir)
	cp zfs/zfs-*.tar.* $(outdir)/
	# Copr passe 'spec' en argument (chemin absolu vers rpm/redhat/*.spec)
	cp zfs/$(spec) $(outdir)/

	# 5) Fabriquer le SRPM *dans le chroot* (macros kmod dispo ici)
	rpmbuild -bs \
	  --define "_sourcedir $(outdir)" \
	  --define "_srcrpmdir $(outdir)" \
	  $(outdir)/$$(basename $(spec))

unexport CC
unexport CXX
unexport CFLAGS
unexport CXXFLAGS
unexport LDFLAGS
unexport MAKEFLAGS

.PHONY: all
all: vinix.iso

.PHONY: prod-all
prod-all:
	cp bootstrap.yml bootstrap.yml.tmp
	sed -i "s/default: 'false' # prod/default: 'true' # prod/g" bootstrap.yml
	$(MAKE) all || true
	mv bootstrap.yml.tmp bootstrap.yml

QEMUFLAGS ?= -M q35,smm=off -m 8G -cdrom vinix.iso -serial stdio

.PHONY: run-kvm
run-kvm: vinix.iso
	qemu-system-x86_64 -enable-kvm -cpu host $(QEMUFLAGS) -smp 4

.PHONY: run-hvf
run-hvf: vinix.iso
	qemu-system-x86_64 -accel hvf -cpu host $(QEMUFLAGS) -smp 4

ovmf:
	mkdir -p ovmf
	cd ovmf && curl -o OVMF-X64.zip https://efi.akeo.ie/OVMF/OVMF-X64.zip && 7z x OVMF-X64.zip

.PHONY: run-uefi
run-uefi: ovmf
	qemu-system-x86_64 -enable-kvm -cpu host $(QEMUFLAGS) -smp 4 -bios ovmf/OVMF.fd

.PHONY: run
run: vinix.iso
	qemu-system-x86_64 $(QEMUFLAGS) -no-shutdown -no-reboot -d int -smp 1

build:
	mkdir -p build
	cd build && [ -f bootstrap.link ] || xbstrap init ..

.PHONY: distro
distro: build
	cd build && xbstrap install -u --all

.PHONY: kernel
kernel: build
	cd build && xbstrap install --rebuild kernel

.PHONY: init
init: build
	cd build && xbstrap install --rebuild init

.PHONY: base-files
base-files: build
	cd build && xbstrap install --rebuild base-files

vinix.iso: build kernel init base-files
	cd build && xbstrap run make-basic-iso
	mv build/vinix.iso ./

.PHONY: clean
clean:
	$(MAKE) -C kernel clean
	rm -f init/init
	rm -f vinix.iso

.PHONY: distclean
distclean: clean
	rm -rf 3rdparty build initramfs.tar.gz pack ovmf
	rm -f kernel/*.xbstrap init/*.xbstrap base-files/*.xbstrap
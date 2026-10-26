# Kernel config based on: arch/arm64/configs/sdm660_defconfig
maintainer="Alexey Minnekhanov <alexeymin@postmarketos.org>"
_flavor="postmarketos-qcom-sdm660"
pkgname=linux-$_flavor
pkgver=7.0.14
pkgrel=61
pkgdesc="Close to mainline linux kernel for Qualcomm Snapdragon 660 SoC"
arch="aarch64"
url="https://kernel.org/"
license="GPL-2.0-only"
options="!strip !check !tracedeps
	pmb:cross-native
	pmb:kconfigcheck-community
	"
makedepends="
	bison
	clang
	diffutils
	elfutils-dev
	findutils
	flex
	linux-headers
	lld
	llvm
	openssl-dev
	pahole
	perl
	postmarketos-installkernel
	python3
	zstd
	"

_tag="v$pkgver-sdm660"

source="
	linux-$_tag.tar.gz::https://github.com/sdm660-mainline/linux/archive/refs/tags/$_tag.tar.gz
	config-$_flavor.aarch64
	01-ipa2-lite-driver.patch
	02-dts-ipa.patch
	03-ipa2-lite-mainline-data-port.patch
	04-sdm660-ipa-v26-hw-init.patch
	05-sdm660-ipa-bus-and-filter-ownership.patch
	06-sdm660-ipa-enable-data-path.patch
	07-sdm660-ipa-qmap-and-filter-order.patch
	08-sdm660-ipa-handle-uc-event-irq.patch
	09-sdm660-ipa-match-downstream-tx-and-ssr.patch
	10-sdm660-ipa-sync-bam-and-irqs.patch
	11-sdm660-ipa-match-vendor-qmi-zip-bounds.patch
	12-sdm660-ipa-match-downstream-noirq-tx.patch
	13-sdm660-ipa-complete-system-pipes.patch
	14-sdm660-ipa-fix-cleanup-lifetime.patch
	15-sdm660-ipa-capture-bam-hresp-fault.patch
	16-sdm660-ipa-use-translated-ap-domain.patch
	17-sdm660-ipa-init-system-header-and-guard-ssr.patch
	18-sdm660-ipa-install-ap-uplink-filters.patch
	19-sdm660-ipa-handle-legacy-filter-enums.patch
	20-sdm660-ipa-reapply-filters-after-data-call.patch
	21-sdm660-ipa-make-family-guard-unconditional.patch
	22-sdm660-ipa-use-modem-owned-embedded-filters.patch
	23-qcom-pon-program-ps-hold-for-poweroff.patch
	24-sdm660-ipa-reenable-ap-filters-after-q6-setup.patch
	25-sdm660-ipa-emit-real-unconditional-catchall.patch
	26-sdm660-ipa-reassert-filter-before-uplink.patch
	27-sdm660-ipa-match-tx-desc-fifo-size.patch
	28-sdm660-ipa-sync-tx-buffer.patch
"

_carch="arm64"

# for some reason, the directory inside .tar.gz does not match tag name: leading
#     "v" is missing, so we can't use $_tag here
builddir="$srcdir/linux-$pkgver-sdm660"

prepare() {
	default_prepare
	cp -v "$srcdir/config-$_flavor.$CARCH" "$builddir"/.config
}

build() {
	unset LDFLAGS

	make ARCH="$_carch" LLVM=1 \
		KBUILD_BUILD_VERSION="$((pkgrel + 1 ))-$_flavor" all Image.gz
}

package() {
	mkdir -p "$pkgdir"/boot

	make modules_install dtbs_install \
		ARCH="$_carch" \
		LLVM=1 \
		INSTALL_PATH="$pkgdir"/boot \
		INSTALL_MOD_PATH="$pkgdir"/usr \
		INSTALL_MOD_STRIP=1 \
		INSTALL_DTBS_PATH="$pkgdir/boot/dtbs-tmp"

	install -D "$builddir/arch/arm64/boot/vmlinuz.efi" \
		"$pkgdir/boot/vmlinuz.efi"

	install -D "$builddir/arch/arm64/boot/Image.gz" \
		"$pkgdir/boot/vmlinuz"

	# Reduce package size, install only qcom SDM630/636/660 dtbs.
	# This reduces .apk size from ~46 Mb to ~30 Mb.
	mkdir -p "$pkgdir"/boot/dtbs/qcom
	cp -r "$pkgdir"/boot/dtbs-tmp/qcom/sda660* "$pkgdir"/boot/dtbs/qcom
	cp -r "$pkgdir"/boot/dtbs-tmp/qcom/sdm630* "$pkgdir"/boot/dtbs/qcom
	cp -r "$pkgdir"/boot/dtbs-tmp/qcom/sdm636* "$pkgdir"/boot/dtbs/qcom
	cp -r "$pkgdir"/boot/dtbs-tmp/qcom/sdm660* "$pkgdir"/boot/dtbs/qcom
	rm -r "$pkgdir"/boot/dtbs-tmp

	# "build" is symlink that will be invalid in rootfs (points to /home/pmos...)
	rm -f "$pkgdir"/usr/lib/modules/*/build "$pkgdir"/usr/lib/modules/*/source

	install -D "$builddir/include/config/kernel.release" \
		"$pkgdir/usr/share/kernel/$_flavor/kernel.release"
}

sha512sums="
83f7e717b8adc9049e828a27629e7347be2fa0aef6c8404608059410cebaafa0c2659317b615634fbd5d83f9a245bcf1a64e5d2cd0d8e3ec8aeaa3dcf0e51298  linux-v7.0.14-sdm660.tar.gz
a709305011f30e4be42ea624c13d8e2af3d60d9a6a0de6856d2061f113c869222022a4668eb63f1b475a6880b079fb8260d3730d6ce70dc7483b60a533b2e12c  config-postmarketos-qcom-sdm660.aarch64
80128383be1545869e91df99ab80d42f11c3e8fb9eb48ebbc86464412112ec14e7fbe6a0e18cbb1dadca39edac76cd9af58c53c4d7560049318a83017d342206  01-ipa2-lite-driver.patch
28588bcbbf1acfab5c18acabf3091be3f4eda32e931a33dc15fa1d449a8914db51d11e77850febaf593a654658de4d808b08e2d20963abf29ec63577b90cdb23  02-dts-ipa.patch
e3e5b78651853724294f8e0a5b04c94142c4d026c064067f1028d69528c1c86e65c67ae8e74580b554d7646795b43e5c5643a60ac237f804dda2efa08df1b0ed  03-ipa2-lite-mainline-data-port.patch
ad218b577f52b11780d8d7c1ea8c5f70af5293182ed008c6cc4e9ab32919f48481011c1488adb3b4c65157d5b3f3bd2b3ec307a8acaf00ab137a795fbcb6f638  04-sdm660-ipa-v26-hw-init.patch
cfa1acfe416ccba9d702650243737d3a1666b5e2908836fed1a567e136a45179fdfc9981be9b2c18b541a3d00cf6b4876a8a12956a92be8432e098777a48d16c  05-sdm660-ipa-bus-and-filter-ownership.patch
f586bd776ec47027fe30687838d459eff96af1dce19a7b91b216056d5777e5bf54e9a6d4dab2a724f35ab4b9377a56981e7a0ea75a9a950db3b495312694d6fc  06-sdm660-ipa-enable-data-path.patch
4f6bbba2ac64c5959b00c4a9472624a850d715c53887c5fb56e8046b89e93bce4b305c4599596e28200a41fb0f2097cd1861a3a692f711960ae696da95a145b9  07-sdm660-ipa-qmap-and-filter-order.patch
692853057fdea040f3257414566fe47055b7ebaae2c3a5c4b5d56b92348027976331bd27dfc3435f1c2373846a0b58f41059ab94efb8703b5136bd8346647317  08-sdm660-ipa-handle-uc-event-irq.patch
69a17d1098a87f2fcea3dd5dea75eb4c676fa0bac6a947c26b31e06d509f1a61c097d0ed891726ab1e70285f95404e9ea1d553b46cb4e60e65434904033dd9c2  09-sdm660-ipa-match-downstream-tx-and-ssr.patch
49547eba4989079bd5bebc9ce07f57c22ab2987941f6615f1245c7dcf558e6f547a5465cd163ad737c919d30a87e39c2d0a81c1b0f8ec6ca061ef0e0d94586cc  10-sdm660-ipa-sync-bam-and-irqs.patch
cf056ae5b918d7dccbcf4ff0f2ed59cb18f5e67a62c57f7157d9f323a0d83e3eef396e8352f171341a17d000037df671230b6b2e09dab8f7f5a2229e262359fd  11-sdm660-ipa-match-vendor-qmi-zip-bounds.patch
0c03791cdcf7fdae37fb8477d8dfeebfcbc8f3f7186ae596e0f1e70901f99ec5a1598053ae7fed098cd5aa4aaadb65869a726224e3fcea556c507e65c11f6bb0  12-sdm660-ipa-match-downstream-noirq-tx.patch
7d7bebb66105b091262579efdeca06becfc11bff508cb89f6659be902d6f43f8761c5b7fe48c658e2646388e2a5109f057eb22e202f54770d0a1f55bf63a34c3  13-sdm660-ipa-complete-system-pipes.patch
802faff2c0ae91804d2046cf979d275c784d8480825f62c7d0fde89128c86744d21cbc78c2393c14390781f51723074bdbcaa942243af68903499d3628a4d0e8  14-sdm660-ipa-fix-cleanup-lifetime.patch
9b6c2adcccf94adc6ec76c75b19dad8e80a71bcb2ce2c9d34b833d639465dbeeef544bffcda6aa48b9942d33bc62f23a6690a17f20bfeb1c3593f66d5e260d85  15-sdm660-ipa-capture-bam-hresp-fault.patch
6e005dcece0dc498ff2ca416e140816957217441595b442e8f42940c69dde52240ab09dbba391e0c6d140a1d194a176d919094665e7d16df4df6b296519a2d8a  16-sdm660-ipa-use-translated-ap-domain.patch
c0f8c5c57da50f0fe86ecc9d0947198c2f41d533b414ff5fe3ff3f3d67e72bab55ab2b9491ab984dea8c3cbc9102bef9751663b7088d9d76d97cfb74c3572b54  17-sdm660-ipa-init-system-header-and-guard-ssr.patch
e53432528138868ae2ce99af8ee1906bc9bf39ccb774d16d214f13dfc000c5b23f8ab67281af8d387ddc3241364033adef6e1e6df0cc5433f0aa53a5cacfe3bf  18-sdm660-ipa-install-ap-uplink-filters.patch
c5d60a8f535db277d35be5556847c83ba94a4b7798e7e0cafd4242b18d482f4c373d78aaa79bcae25a228334782480b4890647ca50c743a40d82882cce749971  19-sdm660-ipa-handle-legacy-filter-enums.patch
c135e5465ec5fc8e8647ecb673d91fdc2570491f377220bf6c498de14e207be0c4f5128c8b84c1888340645bcec76dc481556b741943f5e6d89d7e1966860393  20-sdm660-ipa-reapply-filters-after-data-call.patch
e5c3873f5988e08f5950d3c97ed94573214b193e10db4b60dace0532bd59af22b905839256814769a9d98bd0470da8e931638bfe7163f3144471f030b9a3bdf9  21-sdm660-ipa-make-family-guard-unconditional.patch
25d8c4582ff285f0a2af7042dd0de1db4dec3965950a0d2397c6b1e1e5fc6d33325df089b3089f4ffe0e8df8e5e4bcb28eb911875d1b132509bc19d078d73baf  22-sdm660-ipa-use-modem-owned-embedded-filters.patch
1ae2b0651e2113e5b36c68b53d5ca20a03e91c48b0fe2291cfd92d31a44db781d78fba346f4cccac97cb759a5692dbebf7d704e2010555a34b21686330dd244f  23-qcom-pon-program-ps-hold-for-poweroff.patch
f496270c4f094d4fa8821cb38126b3ced8691278e3e1396e11227fa68e91a310922b5e60445e492c707c3f9898ba4fb52f4326b8d74e6a07549dbd70b34eb5d5  24-sdm660-ipa-reenable-ap-filters-after-q6-setup.patch
4443b68ad36017e6c5d70840dbbbf0164bc35daaa5d942f14a779fe45d53afd806bef9b7790d40a6840957ecd1af3abffa014d0f19d9e91770ecc54b0764a0cc  25-sdm660-ipa-emit-real-unconditional-catchall.patch
59c3eb9f6dace9f685e1b014b1442e46d65287e77d7a3b811ed3505080b31bc5bd06c5205149f5da752565b232560d5c457ae2b84d1c9658cd45d8c7c48a1944  26-sdm660-ipa-reassert-filter-before-uplink.patch
873f1284f9c6c5482d88ce788091ae1c23c89a250f05429fffc061129bbae8027c6b6bc311b308545378f915608e3309097da753cfcdc510129a3deca8d99ba8  27-sdm660-ipa-match-tx-desc-fifo-size.patch
aa0020a0a0e33ae483673b7b6964e148edd4af6322887f5e0a0c36ff9d3af97eb6fd8e96ae12f656d42b685caef9d437702f8468607d32f1e8bd490b18eac224  28-sdm660-ipa-sync-tx-buffer.patch
"

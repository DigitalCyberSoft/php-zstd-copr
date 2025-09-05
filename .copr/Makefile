# COPR Makefile for automated php-zstd builds from git master
# Builds PHP extension from latest git commit

# COPR provides outdir, set default for local testing
outdir ?= .

# Default target for COPR
srpm:
	@echo "=== Starting php-zstd SRPM build from GitHub master ==="
	@echo "Output directory: $(outdir)"
	@echo "Working directory: $$(pwd)"
	@echo "User: $$(whoami)"

	# Install git if not available
	@if ! command -v git >/dev/null 2>&1; then \
		echo "=== Git not found, installing it ==="; \
		if command -v dnf >/dev/null 2>&1; then \
			echo "Using DNF to install git..."; \
			dnf install -y git || (echo "ERROR: Failed to install git with dnf" && exit 1); \
		elif command -v yum >/dev/null 2>&1; then \
			echo "Using YUM to install git..."; \
			yum install -y git || (echo "ERROR: Failed to install git with yum" && exit 1); \
		else \
			echo "ERROR: No package manager found to install git"; \
			exit 1; \
		fi; \
		echo "Git installed successfully"; \
	else \
		echo "=== Git is already available ==="; \
	fi

	# Verify git is now available
	@which git || (echo "ERROR: git still not found after installation attempt" && exit 1)

	# Clone php-ext-zstd from GitHub
	@echo "=== Cloning php-ext-zstd from GitHub ==="; \
	rm -rf php-ext-zstd-master; \
	git clone --depth=50 https://github.com/kjdev/php-ext-zstd.git php-ext-zstd-master || exit 1; \
	\
	echo "=== Getting php-ext-zstd version info ==="; \
	cd php-ext-zstd-master && \
	COMMIT=$$(git rev-parse HEAD); \
	SHORTCOMMIT=$$(git rev-parse --short HEAD); \
	VERSION=$$(sed -n '/#define PHP_ZSTD_VERSION/{s/.* "//;s/".*$$//;p}' php_zstd.h); \
	if [ -z "$$VERSION" ]; then \
		VERSION=$$(git describe --tags --abbrev=0 2>/dev/null | sed 's/^v//') || VERSION="0.13.3"; \
	fi; \
	echo "Commit: $$COMMIT"; \
	echo "Short commit: $$SHORTCOMMIT"; \
	echo "Version: $$VERSION"; \
	cd ..; \
	\
	echo "=== Getting PHP version ==="; \
	PHP_VER=$$(rpm -q --qf '%{version}\n' php-devel 2>/dev/null | head -1 || echo "8.3.0"); \
	PHP_MAJOR=$$(echo $$PHP_VER | cut -d. -f1); \
	PHP_MINOR=$$(echo $$PHP_VER | cut -d. -f2); \
	echo "PHP Version: $$PHP_VER ($$PHP_MAJOR.$$PHP_MINOR)"; \
	\
	echo "=== Determining release number ==="; \
	FULL_VERSION="$$VERSION"_"$$SHORTCOMMIT"; \
	FEDORA_VER=$$(rpm -E %fedora 2>/dev/null || echo "42"); \
	echo "Checking COPR for existing builds of version $$FULL_VERSION on Fedora $$FEDORA_VER"; \
	COPR_URL="https://download.copr.fedorainfracloud.org/results/reversejames/php-zstd/fedora-$$FEDORA_VER-x86_64"; \
	BUILD_DIRS=$$(curl -s "$$COPR_URL/" 2>/dev/null | grep -oE '[0-9]+-php-zstd' | head -10); \
	EXISTING_RELEASES=""; \
	for dir in $$BUILD_DIRS; do \
		RPMS=$$(curl -s "$$COPR_URL/$$dir/" 2>/dev/null | grep -oE "php-zstd-$$VERSION-[0-9]+.*git$$SHORTCOMMIT.*\.fc[0-9]+\.x86_64\.rpm" | head -1); \
		if [ -n "$$RPMS" ]; then \
			REL=$$(echo "$$RPMS" | sed "s/.*-$$VERSION-\([0-9]\+\).*\.fc[0-9]\+\.x86_64\.rpm/\1/"); \
			EXISTING_RELEASES="$$EXISTING_RELEASES $$REL"; \
		fi; \
	done; \
	if [ -n "$$EXISTING_RELEASES" ]; then \
		MAX_REL=$$(echo $$EXISTING_RELEASES | tr ' ' '\n' | sort -n | tail -1); \
		RELEASE_NUM=$$(($$MAX_REL + 1)); \
		echo "Found existing release $$MAX_REL, incrementing to $$RELEASE_NUM"; \
	else \
		RELEASE_NUM=1; \
		echo "No existing builds found for $$FULL_VERSION, starting with release 1"; \
	fi; \
	\
	echo "=== Creating php-ext-zstd tarball ==="; \
	tar czf zstd-$$VERSION.tar.gz -C . php-ext-zstd-master --transform "s/^php-ext-zstd-master/zstd-$$VERSION/" || exit 1; \
	\
	echo "=== Creating package.xml ==="; \
	( \
	echo '<?xml version="1.0" encoding="UTF-8"?>'; \
	echo '<package packagerversion="1.10.13" version="2.0" xmlns="http://pear.php.net/dtd/package-2.0">'; \
	echo " <name>zstd</name>"; \
	echo " <channel>pecl.php.net</channel>"; \
	echo " <summary>Zstandard extension</summary>"; \
	echo " <description>PHP extension for compression and decompression with Zstandard library</description>"; \
	echo " <lead>"; \
	echo "  <name>Kamijo</name>"; \
	echo "  <user>kjdev</user>"; \
	echo "  <email>kamijo@example.com</email>"; \
	echo "  <active>yes</active>"; \
	echo " </lead>"; \
	echo " <date>$$(date +%Y-%m-%d)</date>"; \
	echo " <version>"; \
	echo "  <release>$$VERSION</release>"; \
	echo "  <api>$$VERSION</api>"; \
	echo " </version>"; \
	echo " <stability>"; \
	echo "  <release>stable</release>"; \
	echo "  <api>stable</api>"; \
	echo " </stability>"; \
	echo " <license uri=\"https://opensource.org/licenses/MIT\">MIT</license>"; \
	echo " <notes>Built from git commit $$SHORTCOMMIT</notes>"; \
	echo " <contents>"; \
	echo "  <dir name=\"/\">"; \
	echo "   <file name=\"LICENSE\" role=\"doc\" />"; \
	echo "   <file name=\"README.md\" role=\"doc\" />"; \
	echo "  </dir>"; \
	echo " </contents>"; \
	echo " <dependencies>"; \
	echo "  <required>"; \
	echo "   <php>"; \
	echo "    <min>7.0.0</min>"; \
	echo "   </php>"; \
	echo "   <pearinstaller>"; \
	echo "    <min>1.10.0</min>"; \
	echo "   </pearinstaller>"; \
	echo "  </required>"; \
	echo " </dependencies>"; \
	echo " <providesextension>zstd</providesextension>"; \
	echo " <extsrcrelease />"; \
	echo "</package>"; \
	) > package.xml || exit 1; \
	\
	echo "=== Generating spec file ==="; \
	( \
	echo '%bcond_without tests'; \
	echo '%bcond_without apcu'; \
	echo '%bcond_with libzstd'; \
	echo ''; \
	echo '%global pecl_name   zstd'; \
	echo '%global with_zts    0%{!?_without_zts:%{?__ztsphp:1}}'; \
	echo '%global ini_name    40-%{pecl_name}.ini'; \
	echo '%global sources     %{pecl_name}-%{version}'; \
	echo '%global _configure  ../%{sources}/configure'; \
	echo ''; \
	echo 'Summary:       Zstandard extension'; \
	echo 'Name:          php-%{pecl_name}'; \
	echo "Version:       $$VERSION"; \
	echo "Release:       $$RELEASE_NUM.git$$SHORTCOMMIT%{?dist}%{!?nophptag:%(%{__php} -r 'echo \".\".PHP_MAJOR_VERSION.\".\".PHP_MINOR_VERSION;')}"; \
	echo 'License:       MIT AND BSD-3-Clause'; \
	echo 'URL:           https://github.com/kjdev/php-ext-zstd'; \
	echo 'Source0:       zstd-%{version}.tar.gz'; \
	echo 'Source1:       package.xml'; \
	echo ''; \
	echo 'BuildRequires: gcc'; \
	echo 'BuildRequires: php-devel >= 7.0'; \
	echo '%if %{with apcu}'; \
	echo 'BuildRequires: php-pecl-apcu-devel'; \
	echo '%endif'; \
	echo 'BuildRequires: php-pear'; \
	echo '%if %{with libzstd}'; \
	echo 'BuildRequires: libzstd-devel   >= 1.4'; \
	echo 'Requires:      libzstd%{?_isa} >= 1.4'; \
	echo '%else'; \
	echo 'Provides:      bundled(libzstd) = 1.5.5'; \
	echo '%endif'; \
	echo ''; \
	echo 'Requires:      php(zend-abi) = %{php_zend_api}'; \
	echo 'Requires:      php(api) = %{php_core_api}'; \
	echo ''; \
	echo 'Provides:      php-pecl-%{pecl_name}          = %{version}'; \
	echo 'Provides:      php-pecl-%{pecl_name}%{?_isa}  = %{version}'; \
	echo 'Provides:      php-pecl(%{pecl_name})         = %{version}'; \
	echo 'Provides:      php-pecl(%{pecl_name})%{?_isa} = %{version}'; \
	echo ''; \
	echo '%description'; \
	echo 'PHP extension for compression and decompression with Zstandard library.'; \
	echo ''; \
	echo 'Package built for PHP %(%{__php} -r '"'"'echo PHP_MAJOR_VERSION.\".\".PHP_MINOR_VERSION;'"'"')'; \
	echo ''; \
	echo '%package devel'; \
	echo 'Summary:       %{name} developer files (header)'; \
	echo 'Requires:      %{name}%{?_isa} = %{version}-%{release}'; \
	echo 'Requires:      php-devel%{?_isa}'; \
	echo ''; \
	echo '%description devel'; \
	echo 'These are the files needed to compile programs using %{name}.'; \
	echo ''; \
	echo '%prep'; \
	echo '%setup -qc'; \
	echo ''; \
	echo '%{?_licensedir:sed -e '"'"'/LICENSE/s/role=\"doc\"/role=\"src\"/'"'"' -i package.xml}'; \
	echo ''; \
	echo 'cd %{sources}'; \
	echo '%if %{with libzstd}'; \
	echo '# Use the system library'; \
	echo 'rm -r zstd'; \
	echo 'sed -e '"'"'\:\"zstd/:d'"'"' -i ../package.xml'; \
	echo '%else'; \
	echo 'cp zstd/LICENSE LICENSE-libzstd'; \
	echo '%endif'; \
	echo ''; \
	echo '# Sanity check, really often broken'; \
	echo 'extver=$$(sed -n '"'"'/#define PHP_ZSTD_VERSION/{s/.* \"//;s/\".*$$//;p}'"'"' php_zstd.h)'; \
	echo 'if test \"x$${extver}\" != \"x%{version}\"; then'; \
	echo '   : Error: Upstream extension version is $${extver}, expecting %{version}.'; \
	echo '   exit 1'; \
	echo 'fi'; \
	echo 'cd ..'; \
	echo ''; \
	echo 'mkdir NTS'; \
	echo '%if %{with_zts}'; \
	echo 'mkdir ZTS'; \
	echo '%endif'; \
	echo ''; \
	echo '# Drop in the bit of configuration'; \
	echo 'cat << '"'"'EOF'"'"' | tee %{ini_name}'; \
	echo '; Enable '"'"'%{summary}'"'"' extension module'; \
	echo 'extension = %{pecl_name}.so'; \
	echo 'EOF'; \
	echo ''; \
	echo '%build'; \
	echo 'cd %{sources}'; \
	echo '%{__phpize}'; \
	echo ''; \
	echo 'cd ../NTS'; \
	echo '%configure \'; \
	echo '    --with-php-config=%{__phpconfig} \'; \
	echo '%if %{with libzstd}'; \
	echo '    --with-libzstd \'; \
	echo '%endif'; \
	echo '    --with-libdir=%{_lib} \'; \
	echo '    --enable-zstd'; \
	echo 'make %{?_smp_mflags}'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo 'cd ../ZTS'; \
	echo '%configure \'; \
	echo '    --with-php-config=%{__ztsphpconfig} \'; \
	echo '%if %{with libzstd}'; \
	echo '    --with-libzstd \'; \
	echo '%endif'; \
	echo '    --with-libdir=%{_lib} \'; \
	echo '    --enable-zstd'; \
	echo 'make %{?_smp_mflags}'; \
	echo '%endif'; \
	echo ''; \
	echo '%install'; \
	echo '# Install the NTS stuff'; \
	echo 'make -C NTS install INSTALL_ROOT=%{buildroot}'; \
	echo 'install -D -m 644 %{ini_name} %{buildroot}%{php_inidir}/%{ini_name}'; \
	echo ''; \
	echo '# Install XML package description'; \
	echo 'install -D -m 644 package.xml %{buildroot}%{pecl_xmldir}/%{name}.xml'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '# Install the ZTS stuff'; \
	echo 'make -C ZTS install INSTALL_ROOT=%{buildroot}'; \
	echo 'install -D -m 644 %{ini_name} %{buildroot}%{php_ztsinidir}/%{ini_name}'; \
	echo '%endif'; \
	echo ''; \
	echo '# Test & Documentation'; \
	echo 'cd %{sources}'; \
	echo 'for i in $$(grep '"'"'role=\"test\"'"'"' ../package.xml | sed -e '"'"'s/^.*name=\"//;s/\".*$$//'"'"')'; \
	echo 'do install -Dpm 644 $$i %{buildroot}%{pecl_testdir}/%{pecl_name}/$$i'; \
	echo 'done'; \
	echo 'for i in $$(grep '"'"'role=\"doc\"'"'"' ../package.xml | sed -e '"'"'s/^.*name=\"//;s/\".*$$//'"'"')'; \
	echo 'do install -Dpm 644 $$i %{buildroot}%{pecl_docdir}/%{pecl_name}/$$i'; \
	echo 'done'; \
	echo ''; \
	echo '%check'; \
	echo 'cd %{sources}'; \
	echo ''; \
	echo ': Minimal load test for NTS extension'; \
	echo '%{__php} --no-php-ini \'; \
	echo '    --define extension=%{buildroot}%{php_extdir}/%{pecl_name}.so \'; \
	echo '    --modules | grep '"'"'^%{pecl_name}$$'"'"''; \
	echo ''; \
	echo '%if %{with apcu}'; \
	echo ': Check that apcu is aware of zstd serializer'; \
	echo '%{__php} --no-php-ini \'; \
	echo '    --define extension=apcu.so \'; \
	echo '    --define apc.enabled=1 \'; \
	echo '    --define apc.enable_cli=1 \'; \
	echo '    --define extension=%{buildroot}%{php_extdir}/%{pecl_name}.so \'; \
	echo '    --ri apcu | grep '"'"'%{pecl_name}'"'"''; \
	echo '%endif'; \
	echo ''; \
	echo '%if %{with tests}'; \
	echo ': Upstream test suite  for NTS extension'; \
	echo 'TEST_PHP_EXECUTABLE=%{__php} \'; \
	echo 'TEST_PHP_ARGS=\"-n -d extension=%{buildroot}%{php_extdir}/%{pecl_name}.so\" \'; \
	echo '%{__php} -n run-tests.php -q --offline --show-diff'; \
	echo '%endif'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo ': Minimal load test for ZTS extension'; \
	echo '%{__ztsphp} --no-php-ini \'; \
	echo '    --define extension=%{buildroot}%{php_ztsextdir}/%{pecl_name}.so \'; \
	echo '    --modules | grep '"'"'^%{pecl_name}$$'"'"''; \
	echo ''; \
	echo '%if %{with tests}'; \
	echo ': Upstream test suite  for ZTS extension'; \
	echo 'TEST_PHP_EXECUTABLE=%{__ztsphp} \'; \
	echo 'TEST_PHP_ARGS=\"-n -d extension=%{buildroot}%{php_ztsextdir}/%{pecl_name}.so\" \'; \
	echo '%{__ztsphp} -n run-tests.php -q --offline --show-diff'; \
	echo '%endif'; \
	echo '%endif'; \
	echo ''; \
	echo '%files'; \
	echo '%{?_licensedir:%license %{sources}/LICENSE}'; \
	echo '%doc %{pecl_docdir}/%{pecl_name}'; \
	echo '%{pecl_xmldir}/%{name}.xml'; \
	echo '%if ! %{with libzstd}'; \
	echo '%{?_licensedir:%license %{sources}/LICENSE-libzstd}'; \
	echo '%endif'; \
	echo ''; \
	echo '%config(noreplace) %{php_inidir}/%{ini_name}'; \
	echo '%{php_extdir}/%{pecl_name}.so'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '%config(noreplace) %{php_ztsinidir}/%{ini_name}'; \
	echo '%{php_ztsextdir}/%{pecl_name}.so'; \
	echo '%endif'; \
	echo ''; \
	echo '%files devel'; \
	echo '%doc %{pecl_testdir}/%{pecl_name}'; \
	echo '%{php_incldir}/ext/%{pecl_name}'; \
	echo ''; \
	echo '%if %{with_zts}'; \
	echo '%{php_ztsincldir}/ext/%{pecl_name}'; \
	echo '%endif'; \
	echo ''; \
	echo '%changelog'; \
	echo "* $$(date '+%a %b %d %Y') COPR Builder - $$VERSION-$$RELEASE_NUM.git$$SHORTCOMMIT"; \
	echo '- Automated build from GitHub master branch'; \
	echo "- Git commit: $$SHORTCOMMIT"; \
	) > php-zstd.spec || exit 1; \
	echo "Spec file generated"; \
	\
	echo "=== Building SRPM ==="; \
	rpmbuild -bs \
		--define "_sourcedir $$(pwd)" \
		--define "_specdir $$(pwd)" \
		--define "_builddir $$(pwd)" \
		--define "_srcrpmdir $(outdir)" \
		--define "_rpmdir $$(pwd)" \
		php-zstd.spec || exit 1; \
	\
	echo "=== SRPM build complete ==="; \
	ls -la $(outdir)/*.src.rpm

clean:
	@rm -f *.tar.gz *.spec *.src.rpm *.xml *.rpm
	@rm -rf php-ext-zstd-master

.PHONY: srpm clean
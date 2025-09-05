# PHP-Zstd COPR Build Repository

Automated COPR build configuration for [PHP-Zstd](https://github.com/kjdev/php-ext-zstd) - PHP extension for Zstandard compression.

## Overview

This repository contains the COPR build configuration to automatically build RPM packages for the PHP Zstandard extension from the latest git commits on the master branch.

## Features

- **Automated Builds**: Builds directly from the latest php-ext-zstd git master branch
- **Smart Versioning**: Automatically increments release numbers for the same git commit
- **PHP Version Support**: Builds for the system PHP version (7.0+)
- **NTS and ZTS Support**: Builds both non-thread-safe and thread-safe variants
- **Bundled libzstd**: Includes bundled Zstandard library (1.5.5) by default

## COPR Repository

The built packages are available in COPR at:
```
https://copr.fedorainfracloud.org/coprs/YOUR_USERNAME/php-zstd/
```

### Installation

To use packages from this COPR repository:

```bash
# Enable the COPR repository
sudo dnf copr enable YOUR_USERNAME/php-zstd

# Install php-zstd
sudo dnf install php-zstd

# For development headers
sudo dnf install php-zstd-devel

# Restart PHP-FPM if using it
sudo systemctl restart php-fpm
```

## Configuration

The extension is configured via `/etc/php.d/40-zstd.ini`:
```ini
; Enable 'Zstandard extension' extension module
extension = zstd.so
```

## Verification

To verify the extension is loaded:
```bash
php -m | grep zstd
php -i | grep zstd
```

## Build Process

The build process:
1. Clones the latest php-ext-zstd master branch
2. Extracts version from php_zstd.h
3. Checks COPR for existing builds to determine release number
4. Creates source tarball and package.xml
5. Generates RPM spec file dynamically
6. Builds SRPM for COPR

## Local Testing

To test the build locally:
```bash
cd .copr
make srpm
```

## Files

- `.copr/Makefile` - COPR build configuration
- `README.md` - This file

## Features of the Extension

- Compression and decompression using Zstandard algorithm
- Streaming support for large files
- Dictionary compression support
- Compatible with PHP 7.0 and later
- APCu serializer integration (when APCu is available)

## License

The build configuration is provided as-is. PHP-Zstd itself is licensed under the MIT License.
The bundled libzstd library is licensed under BSD-3-Clause.

## Upstream

- PHP-Zstd GitHub: https://github.com/kjdev/php-ext-zstd
- PECL Package: https://pecl.php.net/package/zstd
- Zstandard: https://facebook.github.io/zstd/

## Contributing

Issues and pull requests can be submitted on GitHub.
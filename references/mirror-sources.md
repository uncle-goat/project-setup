# Mirror Sources Reference

This file contains all domestic mirror source configurations for Chinese network environments. Use these mirrors to ensure fast and reliable dependency installation.

---

## npm / yarn / pnpm

### npm

```bash
npm config set registry https://registry.npmmirror.com
```

Verify:
```bash
npm config get registry
```

### yarn (v1)

```bash
yarn config set registry https://registry.npmmirror.com
```

### yarn (v2+/Berry)

Edit `.yarnrc.yml` in project root:
```yaml
npmRegistryServer: "https://registry.npmmirror.com"
```

Or use environment variable:
```bash
YARN_NPM_REGISTRY_SERVER=https://registry.npmmirror.com yarn install
```

### pnpm

```bash
pnpm config set registry https://registry.npmmirror.com
```

---

## pip (Python)

### Temporary use (per install)

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### Global configuration

```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

### pip.conf / pip.ini

Linux/macOS (`~/.pip/pip.conf`):
```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

Windows (`%APPDATA%\pip\pip.ini`):
```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

### Poetry

```bash
poetry source add tsinghua https://pypi.tuna.tsinghua.edu.cn/simple --priority=primary
```

Or edit `pyproject.toml`:
```toml
[[tool.poetry.source]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
priority = "primary"
```

### Conda

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --set show_channel_urls yes
```

---

## apt (Debian/Ubuntu)

Edit `/etc/apt/sources.list` (replace with the appropriate Ubuntu version codename, e.g. `jammy` for 22.04):

```bash
sed -i 's|http://archive.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
sed -i 's|http://security.ubuntu.com|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
apt update
```

For Debian, replace `archive.ubuntu.com` with `deb.debian.org` and use:
```bash
sed -i 's|http://deb.debian.org|https://mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
apt update
```

---

## Maven (Java)

### Global configuration

Edit `~/.m2/settings.xml`:

```xml
<settings>
  <mirrors>
    <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>*</mirrorOf>
      <name>Aliyun Maven Mirror</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>
</settings>
```

### Project-level configuration

Add to `pom.xml`:
```xml
<repositories>
  <repository>
    <id>aliyun</id>
    <url>https://maven.aliyun.com/repository/public</url>
  </repository>
</repositories>

<pluginRepositories>
  <pluginRepository>
    <id>aliyun-plugin</id>
    <url>https://maven.aliyun.com/repository/public</url>
  </pluginRepository>
</pluginRepositories>
```

### Gradle

Edit `build.gradle`:
```groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/spring' }
        mavenCentral()
    }
}
```

For Gradle Kotlin DSL (`build.gradle.kts`):
```kotlin
allprojects {
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/repository/spring") }
        mavenCentral()
    }
}
```

### Gradle wrapper properties

Edit `gradle/wrapper/gradle-wrapper.properties` to use a domestic Gradle distribution:
```properties
distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.5-bin.zip
```

---

## Go

### Global proxy

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```

### Temporary use

```bash
GOPROXY=https://goproxy.cn,direct go mod download
```

### Verify

```bash
go env GOPROXY
```

---

## Rust / Cargo

### rustup

```bash
export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup
export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup
```

Make persistent by adding to `~/.bashrc` or `~/.zshrc`.

### Cargo crates.io mirror

Create or edit `~/.cargo/config.toml`:

```toml
[source.crates-io]
replace-with = 'tuna'

[source.tuna]
registry = "sparse+https://mirrors.tuna.tsinghua.edu.cn/crates.io-index/"
```

Or use USTC mirror:
```toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
```

---

## Ruby (gem)

```bash
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
```

Verify:
```bash
gem sources -l
```

### Bundler

Edit `~/.bundle/config`:
```yaml
---
BUNDLE_MIRROR__HTTPS://RUBYGEMS__ORG/: https://mirrors.tuna.tsinghua.edu.cn/rubygems/
```

---

## PHP (Composer)

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

Project-level:
```bash
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```

---

## NuGet (.NET)

Create or edit `~/.nuget/NuGet/NuGet.Config`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="aliyun" value="https://mirrors.aliyun.com/nuget/v3/index.json" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

---

## Homebrew (macOS)

```bash
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
```

---

## Docker

Edit `/etc/docker/daemon.json`:

```json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

Then restart Docker:
```bash
systemctl restart docker
```

---

## Quick Reference Table

| Package Manager | Mirror | URL |
|---|---|---|
| npm/yarn/pnpm | Taobao (npmmirror) | `https://registry.npmmirror.com` |
| pip | Tsinghua | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| Poetry | Tsinghua | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| Conda | Tsinghua | `https://mirrors.tuna.tsinghua.edu.cn/anaconda/` |
| apt | Tsinghua | `https://mirrors.tuna.tsinghua.edu.cn/ubuntu` |
| Maven | Aliyun | `https://maven.aliyun.com/repository/public` |
| Gradle | Aliyun | `https://maven.aliyun.com/repository/public` |
| Go | Qiniu (goproxy.cn) | `https://goproxy.cn` |
| Rust/Cargo | Tsinghua | `https://mirrors.tuna.tsinghua.edu.cn/rustup` |
| Ruby gem | Tsinghua | `https://mirrors.tuna.tsinghua.edu.cn/rubygems/` |
| PHP Composer | Aliyun | `https://mirrors.aliyun.com/composer/` |
| NuGet (.NET) | Aliyun | `https://mirrors.aliyun.com/nuget/v3/index.json` |
| Homebrew | Tsinghua | `https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/` |
| Docker | Tencent / USTC | See Docker section above |

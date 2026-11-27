# Maintainer: Shadichy <shadichy@blisslabs.org>

pkgname=cros-modp_b64
pkgver=0.0.1
pkgrel=1
pkgdesc='ChromiumOS modp_b64 high-performance Base64 encoder/decoder static library'
arch=('x86_64')
url='https://chromium.googlesource.com/aosp/platform/external/modp_b64'
license=('BSD-3-Clause')
depends=('gcc-libs' 'glibc')
makedepends=('cros-common-mk' 'gn' 'ninja' 'clang' 'lld' 'llvm' 'git')
options=('staticlibs' '!lto')

_local_src_dir=""

_resolve_local_src() {
  _local_src_dir=""
  if [[ -n "${CROS_LOCAL_SRC:-}" ]]; then
    local _base=""
    if _base="$(cd "${startdir:-$PWD}" && cd "${CROS_LOCAL_SRC}" 2>/dev/null && pwd)"; then
      if [[ -d "${_base}/tree/src/third_party/modp_b64" ]]; then
        _local_src_dir="${_base}/tree/src/third_party/modp_b64"
      elif [[ -d "${_base}/src/third_party/modp_b64" ]]; then
        _local_src_dir="${_base}/src/third_party/modp_b64"
      elif [[ -d "${_base}/third_party/modp_b64" ]]; then
        _local_src_dir="${_base}/third_party/modp_b64"
      elif [[ -f "${_base}/modp_b64/BUILD.gn" && -f "${_base}/modp_b64/modp_b64.cc" ]]; then
        _local_src_dir="${_base}/modp_b64"
      fi
    fi
  fi
}

_resolve_local_src
if [[ -n "${_local_src_dir}" ]]; then
  source=()
  sha256sums=()
else
  source=(
    'modp_b64::git+https://chromium.googlesource.com/aosp/platform/external/modp_b64.git#branch=master'
  )
  sha256sums=('SKIP')
fi

prepare() {
  local src_dir="${srcdir}/modp_b64"

  _resolve_local_src
  if [[ -n "${_local_src_dir}" ]]; then
    src_dir="${_local_src_dir}"
  fi

  rm -rf "${srcdir}/work"
  mkdir -p "${srcdir}/work"

  tar -C "${src_dir}" \
    --exclude='./.git' \
    -cf - . | tar -C "${srcdir}/work" -xf -
}

build() {
  cd "${srcdir}/work"
  export CC=clang
  export CXX=clang++
  export AR=llvm-ar

  rm -rf "${srcdir}/out"
  platform2-arch-build \
    --pkg modp_b64 \
    --src "${srcdir}/work" \
    --out "${srcdir}/out" \
    --action deviterate
}

check() {
  local out_default="${srcdir}/out/out/Default"

  test -s "${out_default}/libmodp_b64.a"
  test -s "${out_default}/obj/modp_b64/libmodp_b64.pc"

  rm -rf "${srcdir}/check_stage"
  mkdir -p \
    "${srcdir}/check_stage/usr/include/modp_b64" \
    "${srcdir}/check_stage/usr/lib/pkgconfig"

  install -Dm644 "${out_default}/libmodp_b64.a" \
    "${srcdir}/check_stage/usr/lib/libmodp_b64.a"
  install -Dm644 "${srcdir}/work/modp_b64/modp_b64.h" \
    "${srcdir}/check_stage/usr/include/modp_b64/modp_b64.h"
  install -Dm644 "${out_default}/obj/modp_b64/libmodp_b64.pc" \
    "${srcdir}/check_stage/usr/lib/pkgconfig/libmodp_b64.pc"

  local -a pc_flags
  read -r -a pc_flags < <(
    PKG_CONFIG_SYSROOT_DIR="${srcdir}/check_stage" \
    PKG_CONFIG_PATH="${srcdir}/check_stage/usr/lib/pkgconfig" \
    pkg-config --cflags --libs libmodp_b64
  )
  printf '%s\n' "${pc_flags[@]}" | grep -Fx -- '-lmodp_b64'

  cat > "${srcdir}/check_modp_b64.cc" <<'EOF'
#include <modp_b64/modp_b64.h>
#include <string>

int main() {
  const std::string input = "hello chromiumos";
  std::string encoded(modp_b64_encode_len(input.size()), '\0');
  const size_t enc_len = modp_b64_encode(encoded.data(), input.data(), input.size());
  encoded.resize(enc_len);
  if (encoded != "aGVsbG8gY2hyb21pdW1vcw==") return 1;

  std::string decoded(modp_b64_decode_len(encoded.size()), '\0');
  const size_t dec_len = modp_b64_decode(decoded.data(), encoded.data(), encoded.size());
  if (dec_len != input.size()) return 2;
  decoded.resize(dec_len);
  if (decoded != input) return 3;
  return 0;
}
EOF

  "${CXX:-clang++}" -std=gnu++20 -Wall -Wextra \
    -I"${srcdir}/check_stage/usr/include" \
    "${srcdir}/check_modp_b64.cc" \
    "${pc_flags[@]}" \
    -o "${srcdir}/check_modp_b64"
  "${srcdir}/check_modp_b64"

  "${CXX:-clang++}" -shared -fPIC -std=gnu++20 \
    -I"${srcdir}/check_stage/usr/include" \
    "${srcdir}/check_modp_b64.cc" \
    "${pc_flags[@]}" \
    -fuse-ld=lld \
    -o "${srcdir}/libcheck_modp_b64.so"
}

package() {
  # Deterministic, explicit installation only: no conditional or fallback install logic.
  install -Dm644 "${srcdir}/out/out/Default/libmodp_b64.a" \
    "${pkgdir}/usr/lib/libmodp_b64.a"
  install -Dm644 "${srcdir}/work/modp_b64/modp_b64.h" \
    "${pkgdir}/usr/include/modp_b64/modp_b64.h"
  install -Dm644 "${srcdir}/out/out/Default/obj/modp_b64/libmodp_b64.pc" \
    "${pkgdir}/usr/lib/pkgconfig/libmodp_b64.pc"
  install -Dm644 "${srcdir}/work/LICENSE" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

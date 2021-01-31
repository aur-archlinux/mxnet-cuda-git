# Maintainer: Butui Hu <hot123tea123@gmail.com>

_CUDA_ARCH_LIST="7.5"
_pkgname=mxnet
pkgname=('mxnet-cuda-git')
_pkgver=1.7.0
pkgver=1.7.0
pkgrel=1
pkgdesc='A flexible and efficient library for deep learning'
arch=('x86_64')
url="http://mxnet.io/"
license=('Apache')
depends=(
  'curl'
  'double-conversion'
  'hdf5'
  'intel-mkl'
  'intel-tbb'
  'libjpeg-turbo'
  'protobuf'
  'python-graphviz'
  'python-numpy'
  'python-requests'
  'zeromq'
)
makedepends=(
  'chrpath'
  'cmake'
  'cuda'
  'cudnn'
  'cython'
  'git'
  'gtk3'
  'nccl'
  'opencv'
  'qt5-base'
)
provides=(mxnet=${pkgver})
conflicts=(mxnet)
source=("$_pkgname::git+https://github.com/apache/incubator-mxnet#commit=64f737cdd59fe88d2c5b479f25d011c5156b6a8a"
)

md5sums=('SKIP')


get_pyver() {
  python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}


prepare() {
  cd "$srcdir/$_pkgname"
  git submodule update --init --recursive

  # do not use 3rd party openmp
  rm -rfv "${srcdir}/${_pkgname}/3rdparty/openmp"
  # CUB is now available bundled with CUDA
  rm -rfv "${srcdir}/${_pkgname}/3rdparty/nvidia_cub"
  # the latest cmake set OpenMP_FOUND instead of OPENMP_FOUND
  sed -i 's/OPENMP_FOUND/OpenMP_FOUND/g' "$srcdir/$_pkgname/CMakeLists.txt"

  rm -rf "$srcdir/$_pkgname/build"
  mkdir "$srcdir/$_pkgname/build"
}

build() {
  cmake_opts=(
    -DBUILD_CPP_EXAMPLES:BOOL=OFF
    -DBUILD_TESTING:BOOL=OFF
    -DCMAKE_INSTALL_LIBDIR:PATH=lib
    -DCMAKE_INSTALL_PREFIX:PATH=/usr
    -DCMAKE_SKIP_INSTALL_RPATH:BOOL=ON
    -DINSTALL_DOCUMENTATION:BOOL=OFF
    -DINSTALL_MKLDNN:BOOL=OFF
    -DMKL_USE_SINGLE_DYNAMIC_LIBRARY:BOOL=ON
    -DUSE_BLAS=mkl
    -DUSE_CPP_PACKAGE:BOOL=ON
    -DUSE_CXX14_IF_AVAILABLE:BOOL=ON
    -DUSE_DIST_KVSTORE:BOOL=ON
    -DUSE_GPERFTOOLS:BOOL=OFF
    -DUSE_JEMALLOC:BOOL=OFF
    -DUSE_LAPACK:BOOL=ON
    -DUSE_LIBJPEG_TURBO:BOOL=ON
    -DUSE_MKLDNN:BOOL=ON
    -DUSE_MKL_IF_AVAILABLE:BOOL=ON
    -DUSE_OPENMP:BOOL=ON
    -DUSE_S3:BOOL=ON
)

  # building with CUDA
  cd "${srcdir}/mxnet/build"
  cmake \
    ${cmake_opts[@]} \
    -DCMAKE_C_COMPILER=/opt/cuda/bin/gcc \
    -DCMAKE_CXX_COMPILER=/opt/cuda/bin/g++ \
    -DCMAKE_CUDA_HOST_COMPILER=/opt/cuda/bin/g++ \
    -DMXNET_CUDA_ARCH=${_CUDA_ARCH_LIST} \
    -DUSE_CUDA:BOOL=ON \
    -DUSE_CUDNN:BOOL=ON \
    -DUSE_NCCL:BOOL=ON \
    -DUSE_OPENCV:BOOL=ON \
    ..
  make
  cd ../python
  python setup.py build --with-cython
}

_package() {
  cd "${srcdir}/${pkgname}/build"

  # install mxnet core component
  make DESTDIR="${pkgdir}" install

  if [ -f "${srcdir}/${pkgname}/build/im2rec" ]; then
    install -Dm755 "${srcdir}/${pkgname}/build/im2rec" "${pkgdir}/usr/bin/im2rec"
    chrpath --delete "${pkgdir}/usr/bin/im2rec"
  fi
  find "${pkgdir}" -type f -name '*.so' -exec chrpath --delete {} \;

  # install python binding
  cd "${srcdir}/${pkgname}/python"
  python setup.py install --root="${pkgdir}" --optimize=1 --with-cython --skip-build
  install -Dm644 "${srcdir}/${pkgname}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

  # create neccesarry soft links
  ln -sf '/usr/lib/libmxnet.so' "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/mxnet/libmxnet.so"
  ln -s "/usr/include" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/mxnet/include"

  mv "${pkgdir}/mkldnn" "${pkgdir}/usr/include"

  # remove unwantted files
  rm -rfv "${pkgdir}/usr/mxnet"
  rm -rfv "${pkgdir}/usr/lib/cmake/dnnl"
  rm -rfv "${pkgdir}/usr/lib/cmake/mkldnn"
  rm -rfv "${pkgdir}/usr/share/doc"
  find "${pkgdir}" -name '*.a' -delete
}

package_mxnet-cuda-git() {
  pkgdesc="${pkgdesc} (with CUDA)"
  depends+=(cuda cudnn nccl opencv)
  provides+=(mxnet-cuda=${pkgver})
  conflicts+=(mxnet-cuda)

  _package
}

# vim:set ts=2 sw=2 et:


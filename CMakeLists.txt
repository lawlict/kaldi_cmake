cmake_minimum_required(VERSION 2.8)
set(CMAKE_CXX_STANDARD 11)

project(kaldi)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread -lgfortran -O2 -msse -msse2 -fPIC")

add_definitions(-DHAVE_CUDA)
add_definitions(-DCUDA_API_PER_THREAD_DEFAULT_STREAM)
add_definitions(-DKALDI_DOUBLEPRECISION=0)
add_definitions(-DHAVE_EXECINFO_H=1)
add_definitions(-DHAVE_CXXABI_H)
add_definitions(-DHAVE_OPENBLAS)

set(kaldi_root $ENV{HOME}/kaldi)

foreach(name
    base matrix util feat cudafeat tree gmm transform sgmm2 fstext hmm
    lm decoder lat cudamatrix nnet nnet2 nnet3 ivector chain kws online2 rnnlm
    cudadecoder
    )
    aux_source_directory(${kaldi_root}/src/${name} cxxs)
    list(FILTER cxxs EXCLUDE REGEX ".*-test.cc$")
    set(kaldi_cxxs ${kaldi_cxxs} ${cxxs})
endforeach(name)

add_library(kaldi STATIC ${kaldi_cxxs})

include_directories(
    ${kaldi_root}/src
    ${kaldi_root}/tools/openfst/include
    ${kaldi_root}/tools/OpenBLAS/install/include
    /usr/local/cuda/include
)

target_link_libraries(kaldi
    # -Wl,--whole-archive
    ${kaldi_root}/tools/OpenBLAS/libopenblas.a
    ${kaldi_root}/tools/openfst/lib/libfst.a
    # -Wl,--no-whole-archive
)

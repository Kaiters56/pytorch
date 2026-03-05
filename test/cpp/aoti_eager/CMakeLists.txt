if(NOT BUILD_PYTHON)
  return()
endif()

set(AOTI_EAGER_TEST_ROOT ${TORCH_ROOT}/test/cpp/aoti_eager)

add_executable(test_aoti_eager
  ${TORCH_ROOT}/test/cpp/common/main.cpp
  ${AOTI_EAGER_TEST_ROOT}/kernel_meta_info_test.cpp
)

target_compile_definitions(test_aoti_eager PRIVATE USE_GTEST)

target_include_directories(test_aoti_eager PRIVATE
  ${TORCH_PYTHON_INCLUDE_DIRECTORIES}
)

target_link_libraries(test_aoti_eager PRIVATE
  torch
  torch_python
  Python::Module
  pybind::pybind11
  gtest_main
)

# torch_python leaves Python C API symbols unresolved (resolved at runtime
# by the Python interpreter). The test headers also pull in pybind11 template
# instantiations with Python C API references in the test .o files.
# --allow-shlib-undefined only handles symbols from .so files, so we need
# --unresolved-symbols=ignore-in-object-files to also handle the .o symbols.
if(NOT MSVC AND NOT APPLE)
  target_link_options(test_aoti_eager PRIVATE
    "LINKER:--unresolved-symbols=ignore-in-object-files")
endif()

if(INSTALL_TEST)
  install(TARGETS test_aoti_eager DESTINATION bin)
  if(MSVC AND BUILD_SHARED_LIBS)
    install(FILES $<TARGET_PDB_FILE:test_aoti_eager> DESTINATION bin OPTIONAL)
  endif()
endif()

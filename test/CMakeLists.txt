# Copyright (c) 2020 The Bitcoin Core developers
# Distributed under the MIT software license, see the accompanying
# file COPYING or http://www.opensource.org/licenses/mit-license.php.

include(CTest)

# Custom test targets for convenience, based on
# https://gitlab.kitware.com/cmake/community/-/wikis/doc/tutorials/EmulateMakeCheck.
#
# CTest already provides a "make test" target, but it just runs existing tests
# that were previously built, without building anything itself. Define "make
# tests" here as a custom target to build all available tests and "make check"
# as a custom target to build and run them.
add_custom_target(tests)
add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND} DEPENDS tests)

if(BUILD_TESTING AND TARGET CapnProto::kj-test)
  set_property(SOURCE ${MP_PROXY_HDRS} PROPERTY GENERATED 1)

  add_executable(mptest
    ${MP_PROXY_HDRS}
    mp/test/foo-types.h
    mp/test/foo.h
    mp/test/test.cpp
  )
  include(${PROJECT_SOURCE_DIR}/cmake/TargetCapnpSources.cmake)
  target_capnp_sources(mptest ${CMAKE_CURRENT_SOURCE_DIR} mp/test/foo.capnp
    IMPORT_PATHS ${CMAKE_SOURCE_DIR}/include
  )
  target_include_directories(mptest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
  target_link_libraries(mptest PRIVATE CapnProto::kj-test)
  target_link_libraries(mptest PRIVATE Threads::Threads)

  add_dependencies(tests mptest)
  add_test(NAME mptest COMMAND mptest)
endif()

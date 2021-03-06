# Create a test configuration file by copying the default configuration file
CONFIGURE_FILE(${CMAKE_SOURCE_DIR}/lib/atomspace-test.conf
    ${PROJECT_BINARY_DIR}/lib/atomspace-test.conf)

# build_type is used in AtomSpaceConfig.cmake.in
string(TOUPPER "${CMAKE_BUILD_TYPE}" build_type )

export(EXPORT AtomSpaceTargets
          FILE "${CMAKE_CURRENT_BINARY_DIR}/AtomSpace/AtomSpaceTargets.cmake"
      )
set(ConfigPackageLocation lib/cmake/AtomSpace)
install(EXPORT AtomSpaceTargets
  FILE
    AtomSpaceTargets.cmake
  DESTINATION
    ${ConfigPackageLocation}
)


include(CMakePackageConfigHelpers)
configure_package_config_file(AtomSpaceConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/AtomSpaceConfig.cmake
                              INSTALL_DESTINATION lib/AtomSpace/cmake
                              PATH_VARS CMAKE_INSTALL_PREFIX)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/AtomSpaceConfigVersion.cmake"
  VERSION ${CPACK_PACKAGE_VERSION}
  COMPATIBILITY SameMajorVersion
)
INSTALL(FILES
       ${CMAKE_CURRENT_BINARY_DIR}/AtomSpaceConfigVersion.cmake
       ${CMAKE_CURRENT_BINARY_DIR}/AtomSpaceConfig.cmake
       DESTINATION
       ${ConfigPackageLocation})
 

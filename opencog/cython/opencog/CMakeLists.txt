
# Need to use -fno-strict-aliasing when compiling cython code, in order
# to avoid nasty compiler warnings about aliasing.  Cython explicitly
# performs aliasing, in order to emulate python object inheritance.
# See, for example,
# https://groups.google.com/forum/#!topic/cython-users/JV1-KvIUeIg
#

MACRO(BUILD_CYTHON PY_V)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-strict-aliasing")

   # MESSAGE(STATUS "Setting python RPATH to ${CMAKE_INSTALL_PREFIX}/${PYTHON${PY_V}_SITE_PACKAGES}")
    #SET(CMAKE_SHARED_LINKER_FLAGS
     #       "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath -Wl,${CMAKE_INSTALL_PREFIX}/${PYTHON${PY_V}_SITE_PACKAGES}")

	INCLUDE_DIRECTORIES(
		${PYTHON${PY_V}_INCLUDE_DIRS}
		${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_BINARY_DIR}
		)

	# Cythonizes one .pyx file into a .cpp file
	# Additional arguments are dependencies
	MACRO(CYTHON_ADD_MODULE_PYX name)
		IF(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pxd)
			SET(DEPENDS ${name}.pyx ${name}.pxd)
        ELSE(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pxd)
            SET(DEPENDS ${name}.pyx)
		ENDIF(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pxd)
		# Allow the user to specify dependencies as optional arguments
		SET(DEPENDS ${DEPENDS} ${ARGN})
		ADD_CUSTOM_COMMAND(
            OUTPUT ${name}${PY_V}.cpp
            COMMAND ${CYTHON_EXECUTABLE}
            ARGS -f
				-${PY_V}
				-I ${PROJECT_BINARY_DIR}
				-o ${name}${PY_V}.cpp
				--cplus ${CMAKE_CURRENT_SOURCE_DIR}/${name}.pyx
            DEPENDS ${DEPENDS}
            COMMENT "Cythonizing ${name}.pyx using Python ${PY_V} Libs")
	ENDMACRO(CYTHON_ADD_MODULE_PYX)

	# Use this as a guide:
	# https://github.com/OpenKinect/libfreenect/blob/master/wrappers/python/CMakeLists.txt

	###################### logger ##########################################
	CYTHON_ADD_MODULE_PYX(logger)
	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "logger${PY_V}.cpp"
		"../../util/Logger.h")

	ADD_LIBRARY(logger_cython${PY_V} SHARED
		logger${PY_V}.cpp
	)

    list(APPEND ADDITIONAL_MAKE_CLEAN_FILES
            "logger${PY_BUILD_VERSION}.cpp")

	TARGET_LINK_LIBRARIES(logger_cython${PY_V}
		${COGUTIL_LIBRARY}
		${PYTHON${PY_V}_LIBRARIES}
	)

	SET_TARGET_PROPERTIES(logger_cython${PY_V} PROPERTIES
		PREFIX ""
		OUTPUT_NAME logger
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

    INSTALL(TARGETS logger_cython${PY_V}
            DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)


	###################### atomspace ####################################
	CYTHON_ADD_MODULE_PYX(atomspace
		"atom.pyx" "nameserver.pyx" "truth_value.pyx"
		"atomspace_details.pyx" opencog_atom_types
		"../../truthvalue/TruthValue.h" "../../truthvalue/SimpleTruthValue.h"
		"../../atoms/proto/NameServer.h" "../../atoms/base/Handle.h"
		"../../atomspace/AtomSpace.h"
	)

	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "atomspace${PY_V}.cpp")
	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "atomspace${PY_V}_api.h")

	# opencog.atomspace Python bindings
	ADD_LIBRARY(atomspace_cython${PY_V} SHARED
		Cast.cc
		atomspace${PY_V}.cpp
	)

	# The atomspace_api.h must be generated before PythonEval is compiled, thus
	# this phony-target is created. Why not add_dependencies between PythonEval
	# and atomspace_cython? -> Because that results in a cyclic dependency.
	ADD_CUSTOM_TARGET(py_atomspace_header${PY_V}
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/atomspace${PY_V}.cpp)

	# XXX FIXME Cython should not static link the atom-types, but
	# but should instead dynamic-load them, as needed.
	ADD_DEPENDENCIES(atomspace_cython${PY_V} opencog_atom_types)

	TARGET_LINK_LIBRARIES(atomspace_cython${PY_V}
		${NO_AS_NEEDED}
		attentionbank
		execution
		atomspace
		atomutils
		clearbox
		atomspaceutils
		truthvalue
		atomcore
		${COGUTIL_LIBRARY}
		${PYTHON${PY_V}_LIBRARIES}
		${Boost_THREAD_LIBRARY}
		${Boost_SYSTEM_LIBRARY}
	)

	SET_TARGET_PROPERTIES(atomspace_cython${PY_V} PROPERTIES
		PREFIX ""
		OUTPUT_NAME atomspace
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

	# The cogserver needs the atomspace pxd file to get the Handle, Atom and
	# AtomSpace defintions during compilation. So this must be installed.
	INSTALL (FILES
		__init__.py
		atomspace.pxd
		Cast.h
		DESTINATION "include/opencog/cython/opencog"
	)

    INSTALL(TARGETS atomspace_cython${PY_V}
            DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)

	############################## type constructors #####################
	CYTHON_ADD_MODULE_PYX(type_constructors
		"atomspace.pxd" opencog_atom_types )

	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "type_constructors${PY_V}.cpp")

	ADD_LIBRARY(type_constructors${PY_V} SHARED
		type_constructors${PY_V}.cpp
	)

	TARGET_LINK_LIBRARIES(type_constructors${PY_V}
		atomspace
		${PYTHON${PY_V}_LIBRARIES}
	)

	SET_TARGET_PROPERTIES(type_constructors${PY_V} PROPERTIES
		PREFIX ""
		OUTPUT_NAME type_constructors
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

    INSTALL(TARGETS type_constructors${PY_V}
            DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)


	############################## utilities #####################
	CYTHON_ADD_MODULE_PYX(utilities
		"atomspace.pxd" opencog_atom_types )

	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "utilities${PY_V}.cpp")

	ADD_LIBRARY(utilities_cython${PY_V} SHARED
		Utilities.cc
		utilities${PY_V}.cpp
	)

	TARGET_LINK_LIBRARIES(utilities_cython${PY_V}
		PythonEval${PY_V}
		clearbox
		atomspace
		type_constructors${PY_V}
		${COGUTIL_LIBRARY}
		${PYTHON${PY_V}_LIBRARIES}
		${Boost_THREAD_LIBRARY}
		${Boost_SYSTEM_LIBRARY}
	)

	SET_TARGET_PROPERTIES(utilities_cython${PY_V} PROPERTIES
		PREFIX ""
		OUTPUT_NAME utilities
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

    INSTALL(TARGETS utilities_cython${PY_V}
            DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)

	############################## bindlink #####################
	CYTHON_ADD_MODULE_PYX(bindlink
		"atomspace.pxd" opencog_atom_types
		"../../query/BindLinkAPI.h")

	list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "bindlink${PY_V}.cpp")

	ADD_LIBRARY(bindlink_cython${PY_V} SHARED
		BindlinkStub.cc
		bindlink${PY_V}.cpp
	)

	TARGET_LINK_LIBRARIES(bindlink_cython${PY_V}
		atomspace_cython${PY_V}
		query
		execution
		${PYTHON${PY_V}_LIBRARIES}
		${Boost_THREAD_LIBRARY}
		${Boost_SYSTEM_LIBRARY}
	)

	SET_TARGET_PROPERTIES(bindlink_cython${PY_V} PROPERTIES
		PREFIX ""
		OUTPUT_NAME bindlink
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

    INSTALL(TARGETS bindlink_cython${PY_V}
            DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)

	IF (HAVE_GUILE)
		############################## scheme wrapper #####################
		CYTHON_ADD_MODULE_PYX(scheme_wrapper
			"PyScheme.h" "atomspace.pxd" opencog_atom_types
			"../load-file.h")

		list(APPEND ADDITIONAL_MAKE_CLEAN_FILES "scheme_wrapper${PY_V}.cpp")

		ADD_LIBRARY(scheme_wrapper${PY_V} SHARED
			PyScheme
			scheme_wrapper${PY_V}.cpp
		)

		TARGET_LINK_LIBRARIES(scheme_wrapper${PY_V}
			atomspace_cython${PY_V}
			query
			execution
			smob
			${PYTHON${PY_V}_LIBRARIES}
		)

		SET_TARGET_PROPERTIES(scheme_wrapper${PY_V} PROPERTIES
			PREFIX ""
			OUTPUT_NAME scheme_wrapper
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python${PY_V})

		INSTALL (TARGETS
			scheme_wrapper${PY_V}
			DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)
	ENDIF (HAVE_GUILE)


	INSTALL (FILES
		__init__.py
		DESTINATION ${PYTHON${PY_V}_SITE_PACKAGES}/opencog)
ENDMACRO()


foreach(PY_V 2 3)
	if (PY_${PY_V}_BINDINGS)
		BUILD_CYTHON(${PY_V})
	endif()
endforeach()



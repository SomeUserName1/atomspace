#
# Need to use -fno-strict-aliasing when compiling cython code, in order
# to avoid nasty compiler warnings about aliasing.  Cython explicitly
# performs aliasing, in order to emulate python object inheritance.
# See, for example, https://groups.google.com/forum/#!topic/cython-users/JV1-KvIUeIg
#
# Also suppress unused function warnings for static functions, since
# Cython generates a static function that it puts in a header which
# causes this warning to appear whenever anyone includes the header
# and doesn't use all the functions.
#

SET(CMAKE_CXX_FLAGS
		"${CMAKE_CXX_FLAGS} -fno-strict-aliasing -Wno-unused-function")

ADD_SUBDIRECTORY (opencog)

# module init
file(MAKE_DIRECTORY opencog)
file(COPY opencog/__init__.py DESTINATION opencog)


MACRO(BUILD_PY_EVAL PY_V)
	INCLUDE_DIRECTORIES(
		${PYTHON${PY_V}_INCLUDE_DIRS}
		${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_BINARY_DIR}
	)

	ADD_LIBRARY(PythonEval${PY_V}
		PythonEval.cc
		load-file.cc
	)

	ADD_DEPENDENCIES(PythonEval${PY_V} py_atomspace_header${PY_V})

	TARGET_LINK_LIBRARIES(PythonEval${PY_V}
		atombase
		atomspace
		${PYTHON${PY_V}_LIBRARIES}
		${Boost_FILESYSTEM_LIBRARY}
	)

	IF(HAVE_GUILE)
		TARGET_LINK_LIBRARIES(PythonEval${PY_V} smob)
	ENDIF(HAVE_GUILE)

	INSTALL (TARGETS PythonEval${PY_V} DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")

	INSTALL (FILES
		PythonEval.h
		PyIncludeWrapper.h
		DESTINATION "include/opencog/cython"
	)

	# ----------------------------------------------------------------
	# The scheme wrapper for PythonEval. Allows python snippets to be
	# called from scheme.
	#
	IF(HAVE_GUILE)
		ADD_LIBRARY(PythonSCM${PY_V}
			PythonSCM.cc
		)
		ADD_DEPENDENCIES(PythonSCM${PY_V} PythonEval${PY_V})

		TARGET_LINK_LIBRARIES(PythonSCM${PY_V}
			PythonEval${PY_V}
			smob
		)

		INSTALL (TARGETS PythonSCM${PY_V} DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")
	ENDIF(HAVE_GUILE)
ENDMACRO()

if (PY_2_BINDINGS)
	ADD_DEFINITIONS(-DCOMPILING_WITH_PY2)
	SET(COMPILING_WITH_PY2 TRUE)
	BUILD_PY_EVAL(2) # add define to import propper agent_finder_api${PY_V}.h
endif()
if (PY_3_BINDINGS)
	SET(COMPILING_WITH_PY2 FALSE)
	BUILD_PY_EVAL(3)
endif()
REMOVE_DEFINITIONS(COMPILING_WITH_PY2 TRUE) # clean up
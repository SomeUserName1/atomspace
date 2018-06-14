
# The atom_types.h file is written to the build directory
INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_BINARY_DIR})



ADD_LIBRARY (execution
	Force.cc
	EvaluationLink.cc
	ExecutionOutputLink.cc
	Instantiator.cc
	MapLink.cc
	ExecSCM.cc
)

# Without this, parallel make will race and crap up the generated files.
ADD_DEPENDENCIES(execution opencog_atom_types)

TARGET_LINK_LIBRARIES(execution
	query
	atomspace
	atombase
	clearbox
	${Boost_SYSTEM_LIBRARY}
)

IF (HAVE_GUILE)
	TARGET_LINK_LIBRARIES(execution smob)
ENDIF (HAVE_GUILE)

FOREACH(PY_V 2 3)
	IF (PY_${PY_V}_BINDINGS)
		INCLUDE_DIRECTORIES(${PYTHON${PY_V}_INCLUDE_PATH})
		TARGET_LINK_LIBRARIES(execution PythonEval${PY_V})
		TARGET_LINK_LIBRARIES(execution ${PYTHON${PY_V}_LIBRARIES})
	ENDIF ()
ENDFOREACH()

INSTALL (TARGETS execution DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")

INSTALL (FILES
	Force.h
	EvaluationLink.h
	ExecutionOutputLink.h
	Instantiator.h
	DESTINATION "include/opencog/atoms/execution"
)

# This does not belong, but cannot be fixed until the Python bindings
# are fixed, first.
INSTALL (FILES
	MapLink.h
	DESTINATION "include/opencog/atoms/core"
)

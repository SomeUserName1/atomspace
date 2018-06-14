
# The atom_types.h file is written to the build directory

MACRO(BUILD_EXECUTION PY_VERSION)
    INCLUDE_DIRECTORIES( ${CMAKE_CURRENT_BINARY_DIR})
    INCLUDE_DIRECTORIES(${PYTHON${PY_VERSION}_INCLUDE_PATH})


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
        PythonEval
        clearbox
        ${Boost_SYSTEM_LIBRARY}
        ${PYTHON${PY_VERSION}_LIBRARIES}
    )

    IF (HAVE_GUILE)
        TARGET_LINK_LIBRARIES(execution smob)
    ENDIF (HAVE_GUILE)


INSTALL (TARGETS execution DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")
ENDMACRO()

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

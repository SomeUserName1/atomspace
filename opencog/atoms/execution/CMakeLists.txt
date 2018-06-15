MACRO(BUILD_EXECUTION PY_VERSION)
    INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
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
        ${Boost_SYSTEM_LIBRARY})

    IF( PY_VERSION != 0 )
        INCLUDE_DIRECTORIES(${PYTHON${PY_VERSION}_INCLUDE_PATH})
        TARGET_LINK_LIBRARIES(${PYTHON${PY_VERSION}_LIBRARIES})
    ENDIF()


    IF (HAVE_GUILE)
        TARGET_LINK_LIBRARIES(execution smob)
    ENDIF (HAVE_GUILE)


    INSTALL (TARGETS execution DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")

    INSTALL (FILES
        Force.h
        EvaluationLink.h
        ExecutionOutputLink.h
        Instantiator.h
        DESTINATION "include/opencog/atoms/execution")

    INSTALL (FILES
        MapLink.h
        DESTINATION "include/opencog/atoms/core")

ENDMACRO()


IF (PY_3_BINDINGS)
    BUILD_EXECUTION(2)
ELSE IF (PY_2_BINDINGS)
    BUILD_EXECUTION(3)
ELSE ()
    BUILD_EXECUTION(0)
ENDIF()

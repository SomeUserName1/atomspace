
ADD_LIBRARY (atomutils
	AtomUtils
	FindUtils
	FuzzyMatch
	FuzzyMatchBasic
	Neighbors
	TypeUtils
)

ADD_DEPENDENCIES(atomutils opencog_atom_types)

TARGET_LINK_LIBRARIES(atomutils
	atombase
	${COGUTIL_LIBRARY}
)

INSTALL (TARGETS atomutils
		DESTINATION "lib${LIB_DIR_SUFFIX}/opencog"
)

INSTALL (FILES
	AtomUtils.h
	FindUtils.h
	FollowLink.h
	ForeachChaseLink.h
	FuzzyMatch.h
	FuzzyMatchBasic.h
	ThreadSafeHandleMap.h
	Neighbors.h
	Substitutor.h
	TypeUtils.h
	DESTINATION "include/opencog/atomutils"
)

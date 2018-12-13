# GVDBX 2018
# Developped By Trigolds
# GVDB library TeatCase 3DPrint CMakeLists
# -----------------------------------------------------------------------------
# ----------------------------- Project configuration -------------------------
# -----------------------------------------------------------------------------

# Environment
SET(3DPrint_EXE GVDB_3DPrint)
SET(3DPrint_EXE ${3DPrint_EXE} PARENT_SCOPE)
Message(STATUS "-------------------------------")
Message(STATUS "Processing TestCase ${3DPrint_EXE}:")
####################################################################################
# Bootstrap
#
SET( SAMPLE_UTILS_DIR ${CONTRIB_ROOT_DIR}/sample_utils )

#####################################################################################
# Sample requirements

set ( REQUIRE_OPENGL "1" )
set ( REQUIRE_PNG "1" )
set ( REQUIRE_TGA "1" )
set ( REQUIRE_GLEW "1" )
set ( REQUIRE_MAIN "1" )
set ( REQUIRE_NVGUI "1" )

####################################################################################
# Add Utils
#
_add_package_Utils()

add_definitions(-DUSE_GVDB)
add_definitions(-DSIM_CODE)
####################################################################################
# Compile PTX Files
#
file(GLOB CUDA_FILES RELATIVE "${SAMPLE_UTILS_DIR}" *.cu *.cuh)
message ( STATUS "Build CUDA kernels: ${CUDA_FILES}" )
get_filename_component( EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR} REALPATH)
_COMPILEPTX ( SOURCES ${CUDA_FILES} TARGET_PATH ${EXECUTABLE_OUTPUT_PATH} GENERATED PTX_FILES INCLUDE "${CMAKE_CURRENT_SOURCE_DIR},${GVDB_INCLUDE_DIR}" OPTIONS -arch=compute_30 -code=sm_30 --ptxas-options=-v -O3 --use_fast_math --maxrregcount=128 )

# #####################################################################################
# # Asset Path
# #
# if ( NOT DEFINED ASSET_PATH )
  # set ( ASSET_PATH ${CMAKE_CURRENT_SOURCE_DIR}/data CACHE PATH "Full path to 3Dprint assets" )  
# endif()
# SET( ASSET_PATH ${CMAKE_CURRENT_SOURCE_DIR}/data)
####################################################################################
# Executable
#
LIST(APPEND ALL_SOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/main_3dprint.cpp ${UTIL_SOURCE_FILES})
include_directories( ${GVDB_INCLUDE_DIR} )
add_definitions(-DGVDB_IMPORTS -DGLEW_STATIC -DGLEW_NO_GLU)  
add_definitions(-DASSET_PATH="${ASSET_PATH}/")
add_executable (${3DPrint_EXE} ${ALL_SOURCE_FILES} )
set_property ( TARGET ${3DPrint_EXE} APPEND PROPERTY DEPENDS ${PTX_SOURCES} )

#####################################################################################
# Additional Libraries
#
# Message(WARNING "GVDB_LIB_DIR: ${GVDB_LIB_DIR}")
# Message(WARNING "GVDB_LIB: ${GVDB_LIB}")
LIST(INSERT GVDBXLinkedLibraries 0 ${GVDB_LIB})
LIST(REMOVE_DUPLICATES GVDBXLinkedLibraries)
# TARGET_LINK_LIBRARIES(${3DPrint_EXE} LINK_PUBLIC ${LIBRARIES_OPTIMIZED} ${GVDBXLinkedLibraries} ${LIBRARIES_DEBUG} ${PLATFORM_LIBRARIES})
_LINK ( TARGET ${3DPrint_EXE} OPT ${LIBRARIES_OPTIMIZED} ${GVDBXLinkedLibraries} DEBUG ${LIBRARIES_DEBUG} PLATFORM ${PLATFORM_LIBRARIES} )

# -- copy dll collection
#
# COLLECT_DDL_FROM_FILES(GVDB_LIB_DIR)
MESSAGE(STATUS "Libraries that will be copied: ${GVDB_DLLS}")
FOREACH( f ${GVDB_DLLS})
  ADD_CUSTOM_COMMAND(TARGET ${3DPrint_EXE} POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${f} $<TARGET_FILE_DIR:${3DPrint_EXE}> )  
ENDFOREACH() 

# -- copy example file
#
SET(example_file ${CMAKE_CURRENT_SOURCE_DIR}/data/lucy.obj)
FOREACH( f ${example_file})
  ADD_CUSTOM_COMMAND(TARGET ${3DPrint_EXE} POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${f} $<TARGET_FILE_DIR:${3DPrint_EXE}> )  
ENDFOREACH() 
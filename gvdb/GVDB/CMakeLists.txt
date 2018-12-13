# GVDBX 2018
# Developped By Trigolds
# GVDB library CMakeLists
# -----------------------------------------------------------------------------
# ----------------------------- Project configuration -------------------------
# -----------------------------------------------------------------------------

SET(GVDB_LIB "${PROJECT_NAME}.GVDB_LIB")
SET(GVDB_LIB ${GVDB_LIB} PARENT_SCOPE)
Message(STATUS "-------------------------------")
Message(STATUS "Processing Library ${GVDB_LIB}:")
#####################################################################################
# Included Cmakes
SET(GVDB_LIB_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
SET(GVDB_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
SET(GVDB_INCLUDE_DIR ${GVDB_INCLUDE_DIR} PARENT_SCOPE)
include_directories ("${CMAKE_CURRENT_SOURCE_DIR}/src")    

#####################################################################################
# GVDB Options
#
set ( USE_BITMASKS OFF CACHE BOOL "Use Bitmasks" )
if ( USE_BITMASKS ) 
   add_definitions(-DUSE_BITMASKS)
endif()

set ( USE_SIMULATION OFF CACHE BOOL "Use GVDB 2.0 Simulation" )
if ( USE_SIMULATION ) 
   add_definitions(-DSIM_CODE)
endif()

set ( USE_OPENVDB OFF CACHE BOOL "Use OpenVDB" FORCE)

set ( USE_NVTX OFF CACHE BOOL "Use NSIGHT Markers" )
if ( USE_NVTX ) 
   add_definitions(-DUSE_NVTX)
endif()

#####################################################################################
# Add CUDPP Library
#
_add_package_CUDPP()
#####################################################################################
# Add OPENVDB Library (optional)
#
_add_package_OpenVDB()

#####################################################################################
# Source files for this project
#
file(GLOB SOURCE_FILES src/*.cpp src/*.hpp src/*.inl src/*.h src/*.c)
#####################################################################################
# Additional files for tools/helpers
#
SET(COMMON_SOURCE_FILES ${GLEW_SOURCE} )

file(GLOB CUDA_FILES RELATIVE "${GVDB_LIB_SOURCE_DIR}" kernels/*.cu kernels/*.cuh )
message ( STATUS "Located CUDA kernels: ${CUDA_FILES}" )

if ( NOT DEFINED CUDA_ARCH )
   SET( CUDA_ARCH "compute_30" CACHE STRING "CUDA Architecture target")
   SET( CUDA_CODE "sm_30" CACHE STRING "CUDA Code target")
endif ()

get_filename_component( LIB_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR} REALPATH)
_COMPILEPTX ( SOURCES ${CUDA_FILES} TARGET_PATH ${LIB_OUTPUT_PATH} GENERATED CUDA_PTX GENPATHS CUDA_PTX_PATHS INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/kernels" OPTIONS -DSIM_CODE -arch=${CUDA_ARCH} -code=${CUDA_CODE} --ptxas-options=-v -O3 --use_fast_math --maxrregcount=64 )

#####################################################################################
# Executable
#
unset ( GVDB_LIB_SOURCE_FILES )
list( APPEND GVDB_LIB_SOURCE_FILES ${SOURCE_FILES} )
list( APPEND GVDB_LIB_SOURCE_FILES ${COMMON_SOURCE_FILES} )
list( APPEND GVDB_LIB_SOURCE_FILES ${PACKAGE_SOURCE_FILES} )

# Definitions
add_definitions(-DGVDB_EXPORTS)					# Export dll symbols

# Set the library type
OPTION (BUILD_SHARED_LIBS "Build Shared Libraries" ON)
set (LIB_TYPE STATIC)
if (BUILD_SHARED_LIBS)
  set (LIB_TYPE SHARED)
endif()

# Build with OpenGL
OPTION (BUILD_OPENGL "Build with OpenGL" ON)
if (BUILD_OPENGL)
	add_definitions(-DBUILD_OPENGL)  				# Build OpenGL
endif()

if( WIN32 AND NOT GLUT_FOUND)
  add_definitions(/wd4267) #remove size_t to int warning
  add_definitions(/wd4996) #remove printf warning
  add_definitions(/wd4244) #remove double to float conversion warning
  add_definitions(/wd4305) #remove double to float truncation warning
  add_library (${GVDB_LIB} ${LIB_TYPE} ${GVDB_LIB_SOURCE_FILES} ${CUDA_FILES} )
  Message(WARNING "GVDB_LIB_SOURCE_FILES: ${GVDB_LIB_SOURCE_FILES}")
  Message(WARNING "CUDA_FILES: ${CUDA_FILES}")
else()
  add_library (${GVDB_LIB} ${LIB_TYPE} ${GVDB_LIB_SOURCE_FILES} ${CUDA_FILES} ${PTX_FILES} )
endif()
SET(GVDB_LIB_DIR ${CMAKE_CURRENT_BINARY_DIR})
SET(GVDB_LIB_DIR ${GVDB_LIB_DIR} PARENT_SCOPE)
#####################################################################################
# Linkage
#
_LINK ( TARGET ${GVDB_LIB} OPT ${LIBRARIES_OPTIMIZED} PLATFORM ${PLATFORM_LIBRARIES} )

# -- copy shaders --
#
FILE( GLOB SHADERS shaders/*.glsl)
MESSAGE(STATUS "Shaders that will be copied: ${SHADERS}")
FOREACH( f ${SHADERS})
  ADD_CUSTOM_COMMAND(TARGET ${GVDB_LIB} POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${f} $<TARGET_FILE_DIR:${GVDB_LIB}> )  
ENDFOREACH() 

LIST(APPEND GVDB_DLLS ${SHADERS})
SET(GVDB_DLLS ${GVDB_DLLS} PARENT_SCOPE)

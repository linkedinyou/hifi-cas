set(EXTERNAL_NAME bullet)

if (WIN32)
  set(PLATFORM_CMAKE_ARGS "-DUSE_MSVC_RUNTIME_LIBRARY_DLL=1")
else ()
  set(PLATFORM_CMAKE_ARGS "-DBUILD_SHARED_LIBS=1")
  
  if (ANDROID)
    list(APPEND PLATFORM_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}" "-DANDROID_NATIVE_API_LEVEL=19")
  elseif (APPLE)
    list(APPEND PLATFORM_CMAKE_ARGS "-DCMAKE_INSTALL_NAME_DIR=<INSTALL_DIR>/lib")
  endif()
endif ()

include(ExternalProject)

if (WIN32)
  if (UPPER_CMAKE_BUILD_TYPE MATCHES DEBUG)
    set(MSBUILD_CONFIGURATION Debug)
  else ()
    set(MSBUILD_CONFIGURATION Release)
  endif ()
  
  find_program(MSBUILD_COMMAND msbuild PATHS "C:/Program Files (x86)/MSBUILD/12.0/Bin")
  
  ExternalProject_Add(
    ${EXTERNAL_NAME}
    URL https://bullet.googlecode.com/files/bullet-2.82-r2704.zip
    URL_MD5 f5e8914fc9064ad32e0d62d19d33d977
    CMAKE_ARGS ${PLATFORM_CMAKE_ARGS} -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DBUILD_EXTRAS=0 -DINSTALL_LIBS=1 -DBUILD_DEMOS=0 -DUSE_GLUT=0
    BUILD_COMMAND ${MSBUILD_COMMAND} ALL_BUILD.vcxproj /p:Configuration=${MSBUILD_CONFIGURATION}
    LOG_DOWNLOAD 1
    LOG_CONFIGURE 1
    LOG_BUILD 1
    BINARY_DIR ${EXTERNAL_PROJECT_PREFIX}/build
  )  
else ()
  ExternalProject_Add(
    ${EXTERNAL_NAME}
    URL http://bullet.googlecode.com/files/bullet-2.82-r2704.tgz
    URL_MD5 70b3c8d202dee91a0854b4cbc88173e8
    CMAKE_ARGS ${PLATFORM_CMAKE_ARGS} -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DBUILD_EXTRAS=0 -DINSTALL_LIBS=1 -DBUILD_DEMOS=0 -DUSE_GLUT=0
    LOG_DOWNLOAD 1
    LOG_CONFIGURE 1
    LOG_BUILD 1
    BINARY_DIR ${EXTERNAL_PROJECT_PREFIX}/build
  )  
endif ()

ExternalProject_Get_Property(${EXTERNAL_NAME} INSTALL_DIR)

string(TOUPPER ${EXTERNAL_NAME} EXTERNAL_NAME_UPPER)

set(BULLET_LIB_DIR "${INSTALL_DIR}/lib")

if (APPLE OR UNIX OR ANDROID)
  if (APPLE)
    set(BULLET_LIB_EXT "dylib")
  else ()
    set(BULLET_LIB_EXT "so")
  endif ()
  
  set(LIB_PREFIX "lib")
elseif (WIN32)
  set(BULLET_LIB_EXT "lib")
endif ()

if (DEFINED BULLET_LIB_EXT)
  if (NOT WIN32 OR UPPER_CMAKE_BUILD_TYPE MATCHES RELEASE)
    set(_PRESENT_LIB_TYPE RELEASE)
    set(_MISSING_LIB_TYPE DEBUG)
  else ()
    set(_PRESENT_LIB_TYPE DEBUG)
    set(_MISSING_LIB_TYPE RELEASE)
    set(_LIB_NAME_SUFFIX _Debug)
  endif ()
  
  set(${EXTERNAL_NAME_UPPER}_DYNAMICS_LIBRARY_${_PRESENT_LIB_TYPE} ${BULLET_LIB_DIR}/${LIB_PREFIX}BulletDynamics${_LIB_NAME_SUFFIX}.${BULLET_LIB_EXT} CACHE FILEPATH "Bullet dynamics ${_PRESENT_LIB_TYPE} library location")
  set(${EXTERNAL_NAME_UPPER}_DYNAMICS_LIBRARY_${_MISSING_LIB_TYPE} "" CACHE FILEPATH "Bullet dynamics ${_MISSING_LIB_TYPE} library location")

  set(${EXTERNAL_NAME_UPPER}_COLLISION_LIBRARY_${_PRESENT_LIB_TYPE} ${BULLET_LIB_DIR}/${LIB_PREFIX}BulletCollision${_LIB_NAME_SUFFIX}.${BULLET_LIB_EXT} CACHE FILEPATH "Bullet collision ${_PRESENT_LIB_TYPE} library location")
  set(${EXTERNAL_NAME_UPPER}_COLLISION_LIBRARY_${_MISSING_LIB_TYPE} "" CACHE FILEPATH "Bullet collision ${_MISSING_LIB_TYPE} library location")

  set(${EXTERNAL_NAME_UPPER}_MATH_LIBRARY_${_PRESENT_LIB_TYPE} ${BULLET_LIB_DIR}/${LIB_PREFIX}LinearMath${_LIB_NAME_SUFFIX}.${BULLET_LIB_EXT} CACHE FILEPATH "Bullet math ${_PRESENT_LIB_TYPE} library location")
  set(${EXTERNAL_NAME_UPPER}_MATH_LIBRARY_${_MISSING_LIB_TYPE} "" CACHE FILEPATH "Bullet math ${_MISSING_LIB_TYPE} library location")

  set(${EXTERNAL_NAME_UPPER}_SOFTBODY_LIBRARY_${_PRESENT_LIB_TYPE} ${BULLET_LIB_DIR}/${LIB_PREFIX}BulletSoftBody${_LIB_NAME_SUFFIX}.${BULLET_LIB_EXT} CACHE FILEPATH "Bullet softbody ${_PRESENT_LIB_TYPE} library location")
  set(${EXTERNAL_NAME_UPPER}_SOFTBODY_LIBRARY_${_MISSING_LIB_TYPE} "" CACHE FILEPATH "Bullet softbody ${_MISSING_LIB_TYPE} library location")
endif ()

if (DEFINED ${EXTERNAL_NAME_UPPER}_DYNAMICS_LIBRARY_${_PRESENT_LIB_TYPE})
  set(${EXTERNAL_NAME_UPPER}_INCLUDE_DIR ${INSTALL_DIR}/include/bullet CACHE PATH "Path to bullet include directory")
endif ()
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.


find_package(Python3 COMPONENTS Development)

find_package(CUDA)

if ((NOT GPU) OR (NOT Python3_FOUND))
    set (CUDA_FOUND false)
endif()

if (CUDA_FOUND)
    message (STATUS "Found cuda.")
else()
    message (STATUS "Could not find cuda.")
endif()

if (Python3_FOUND)
    include_directories (${Python3_INCLUDE_DIRS})
    link_directories (${Python3_LIBRARY_DIRS})
    set (Python_INCLUDE_DIRS ${Python3_INCLUDE_DIRS})
    set (Python_LIBRARIES ${Python3_LIBRARIES})
    set (Python_FOUND true)
else()
    find_package(Python2 COMPONENTS Development)
    if (Python2_FOUND)
        include_directories (${Python2_INCLUDE_DIRS})
        link_directories (${Python2_LIBRARY_DIRS})
        set (Python_INCLUDE_DIRS ${Python2_INCLUDE_DIRS})
        set (Python_LIBRARIES ${Python2_LIBRARIES})
        set (Python_FOUND true)
    endif()
endif()

if (Python_FOUND)
    message (STATUS "Found Python.")
    message (STATUS "Include Path: ${Python_INCLUDE_DIRS}")
    message (STATUS "Library Path: ${Python_LIBRARIES}")

    if (WIN32)
        set(PY_SUFFIX .pyd)
    else()
        set(PY_SUFFIX .so)
    endif()

    execute_process(COMMAND swig -python -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/CoreInterface_pwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/PythonCore.i)
    execute_process(COMMAND swig -python -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/ClientInterface_pwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/PythonClient.i)

    include_directories(${PYTHON_INCLUDE_PATH} ${PROJECT_SOURCE_DIR}/AnnService ${PROJECT_SOURCE_DIR}/Wrappers)

    file(GLOB CORE_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/CoreInterface.h)
    file(GLOB CORE_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/CoreInterface.cpp ${CMAKE_BINARY_DIR}/CoreInterface_pwrap.cpp)
    add_library (_SPTAG SHARED ${CORE_SRC_FILES} ${CORE_HDR_FILES})
    set_property(TARGET _SPTAG PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/CoreInterface_pwrap.cpp ${CMAKE_BINARY_DIR}/SPTAG.py)
    set_target_properties(_SPTAG PROPERTIES PREFIX "" SUFFIX ${PY_SUFFIX})
    if (CUDA_FOUND)
        target_link_libraries(_SPTAG GPUSPTAGLibStatic ${Python_LIBRARIES})
    else()
        target_link_libraries(_SPTAG SPTAGLibStatic ${Python_LIBRARIES})
    endif()

    file(GLOB CLIENT_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/ClientInterface.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Socket/*.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Client/*.h)
    file(GLOB CLIENT_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/ClientInterface.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Socket/*.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Client/*.cpp ${CMAKE_BINARY_DIR}/ClientInterface_pwrap.cpp)
    add_library (_SPTAGClient SHARED ${CLIENT_SRC_FILES} ${CLIENT_HDR_FILES})
    set_property(TARGET _SPTAGClient PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/ClientInterface_pwrap.cpp ${CMAKE_BINARY_DIR}/SPTAGClient.py)
    set_target_properties(_SPTAGClient PROPERTIES PREFIX "" SUFFIX ${PY_SUFFIX})
    target_link_libraries(_SPTAGClient SPTAGLibStatic ${Python_LIBRARIES} ${Boost_LIBRARIES})

    install(TARGETS _SPTAG _SPTAGClient
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib)
    install(FILES ${CMAKE_BINARY_DIR}/SPTAG.py ${CMAKE_BINARY_DIR}/SPTAGClient.py DESTINATION bin)
else()
    message (STATUS "Could not find Python.")
endif()

find_package(JNI)
if (JNI_FOUND)
    include_directories (${JNI_INCLUDE_DIRS})
    link_directories (${JNI_LIBRARY_DIRS})
    message (STATUS "Found JNI.")
    message (STATUS "Include Path: ${JNI_INCLUDE_DIRS}")
    message (STATUS "Library Path: ${JNI_LIBRARIES}")

    if (WIN32)
        set (JAVA_SUFFIX .dll)
    else()
        set (JAVA_SUFFIX .so)
    endif()

    execute_process(COMMAND swig -java -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/CoreInterface_jwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/JavaCore.i)
    execute_process(COMMAND swig -java -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/ClientInterface_jwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/JavaClient.i)

    include_directories(${JNI_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/AnnService ${PROJECT_SOURCE_DIR}/Wrappers)

    file(GLOB CORE_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/CoreInterface.h)
    file(GLOB CORE_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/CoreInterface.cpp ${CMAKE_BINARY_DIR}/CoreInterface_jwrap.cpp)
    add_library (JAVASPTAG SHARED ${CORE_SRC_FILES} ${CORE_HDR_FILES})
    set_property(TARGET JAVASPTAG PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/CoreInterface_jwrap.cpp ${CMAKE_BINARY_DIR}/*.java)
    set_target_properties(JAVASPTAG PROPERTIES SUFFIX ${JAVA_SUFFIX})
    target_link_libraries(JAVASPTAG SPTAGLibStatic ${JNI_LIBRARIES})

    file(GLOB CLIENT_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/ClientInterface.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Socket/*.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Client/*.h)
    file(GLOB CLIENT_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/ClientInterface.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Socket/*.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Client/*.cpp ${CMAKE_BINARY_DIR}/ClientInterface_jwrap.cpp)
    add_library (JAVASPTAGClient SHARED ${CLIENT_SRC_FILES} ${CLIENT_HDR_FILES})
    set_property(TARGET JAVASPTAGClient PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/ClientInterface_jwrap.cpp ${CMAKE_BINARY_DIR}/*.java)
    set_target_properties(JAVASPTAGClient PROPERTIES SUFFIX ${JAVA_SUFFIX})
    target_link_libraries(JAVASPTAGClient SPTAGLibStatic ${JNI_LIBRARIES} ${Boost_LIBRARIES})

    install(TARGETS JAVASPTAG JAVASPTAGClient
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib)
    install(FILES ${CMAKE_BINARY_DIR}/*.java DESTINATION bin)
else()
    message (STATUS "Could not find JNI.")
endif()

if (WIN32)
    if (${PROJECTNAME_ARCHITECTURE} MATCHES "x64")
        set (csharp_dotnet_framework_hints "$ENV{windir}\\Microsoft.NET\\Framework64")
    else()
        set (csharp_dotnet_framework_hints "$ENV{windir}\\Microsoft.NET\\Framework")
    endif()

    file(GLOB_RECURSE csharp_dotnet_executables ${csharp_dotnet_framework_hints}/csc.exe)
    list(SORT csharp_dotnet_executables)
    list(REVERSE csharp_dotnet_executables)
    foreach (csharp_dotnet_executable ${csharp_dotnet_executables})
        if (NOT DEFINED DOTNET_FOUND)
            string(REPLACE "${csharp_dotnet_framework_hints}/" "" csharp_dotnet_version_temp ${csharp_dotnet_executable})
            string(REPLACE "/csc.exe" "" csharp_dotnet_version_temp ${csharp_dotnet_version_temp})

            set (DOTNET_EXECUTABLE_VERSION "${csharp_dotnet_version_temp}" CACHE STRING "C# .NET compiler version" FORCE)
            set (DOTNET_FOUND ${csharp_dotnet_executable})
        endif()
    endforeach(csharp_dotnet_executable)
else()
    FIND_PROGRAM(DOTNET_FOUND dotnet)
endif()

if (DOTNET_FOUND)
    message (STATUS "Found dotnet.")
    message (STATUS "DOTNET_EXECUTABLE: " ${DOTNET_FOUND})

    if (WIN32)
        set (CSHARP_SUFFIX .dll)
    else()
        set (CSHARP_SUFFIX .so)
    endif()

    execute_process(COMMAND swig -csharp -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/CoreInterface_cwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/CsharpCore.i)
    execute_process(COMMAND swig -csharp -c++ -I${PROJECT_SOURCE_DIR}/Wrappers/inc -o ${CMAKE_BINARY_DIR}/ClientInterface_cwrap.cpp ${PROJECT_SOURCE_DIR}/Wrappers/inc/CsharpClient.i)

    include_directories(${PROJECT_SOURCE_DIR}/AnnService ${PROJECT_SOURCE_DIR}/Wrappers)

    file(GLOB CORE_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/CoreInterface.h)
    file(GLOB CORE_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/CoreInterface.cpp ${CMAKE_BINARY_DIR}/CoreInterface_cwrap.cpp)
    add_library (CSHARPSPTAG SHARED ${CORE_SRC_FILES} ${CORE_HDR_FILES})
    set_property(TARGET CSHARPSPTAG PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/CoreInterface_cwrap.cpp ${CMAKE_BINARY_DIR}/*.cs)
    set_target_properties(CSHARPSPTAG PROPERTIES SUFFIX ${CSHARP_SUFFIX})
    target_link_libraries(CSHARPSPTAG SPTAGLibStatic)

    file(GLOB CLIENT_HDR_FILES ${PROJECT_SOURCE_DIR}/Wrappers/inc/ClientInterface.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Socket/*.h ${PROJECT_SOURCE_DIR}/AnnService/inc/Client/*.h)
    file(GLOB CLIENT_SRC_FILES ${PROJECT_SOURCE_DIR}/Wrappers/src/ClientInterface.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Socket/*.cpp ${PROJECT_SOURCE_DIR}/AnnService/src/Client/*.cpp ${CMAKE_BINARY_DIR}/ClientInterface_cwrap.cpp)
    add_library (CSHARPSPTAGClient SHARED ${CLIENT_SRC_FILES} ${CLIENT_HDR_FILES})
    set_property(TARGET CSHARPSPTAGClient PROPERTY ADDITIONAL_CLEAN_FILES ${CMAKE_BINARY_DIR}/ClientInterface_cwrap.cpp ${CMAKE_BINARY_DIR}/*.cs)
    set_target_properties(CSHARPSPTAGClient PROPERTIES SUFFIX ${CSHARP_SUFFIX})
    target_link_libraries(CSHARPSPTAGClient SPTAGLibStatic ${Boost_LIBRARIES})

    install(TARGETS CSHARPSPTAG CSHARPSPTAGClient
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib)
    install(FILES ${CMAKE_BINARY_DIR}/*.cs DESTINATION bin)
else()
    message (STATUS "Could not find C#.")
endif()


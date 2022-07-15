cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(MAIN_APP_PROJECT_NAME
        qltest
        )
set(MAIN_APP_SOURCE_DIR
        src
        )
set(MAIN_APP_SOURCE
        ${MAIN_APP_SOURCE_DIR}/app.cpp
        )
set(MAIN_LIBRARIES_DIR
        ${CMAKE_SOURCE_DIR}/libs
        )
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH ON) # https://stackoverflow.com/questions/47697761/cmake-run-time-error-dyld-library-not-loaded-for-dynamically-linked-resource
# build
project(${MAIN_APP_PROJECT_NAME})
add_executable(${MAIN_APP_PROJECT_NAME} ${MAIN_APP_SOURCE})
add_subdirectory("${MAIN_LIBRARIES_DIR}/quantlib")
target_link_libraries(${MAIN_APP_PROJECT_NAME}
        ql_library
        )
target_include_directories(${MAIN_APP_PROJECT_NAME} PUBLIC
        /usr/local/include
        ${CMAKE_INSTALL_PREFIX}/include
        )

# install
install(TARGETS ${MAIN_APP_PROJECT_NAME} DESTINATION bin)

# gtest
include(FetchContent)
FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)
FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(qltest_test test/app_test.cpp)
target_link_libraries(qltest_test gtest_main)

include(GoogleTest)
gtest_discover_tests(qltest_test)
# Copyright 2015-2018 by Martin Moene
#
# gsl-lite is based on GSL: Guideline Support Library,
# https://github.com/microsoft/gsl
#
# This code is licensed under the MIT License (MIT).

cmake_minimum_required( VERSION 3.0 )

project( test )

set( INCDIRS "${PROJECT_SOURCE_DIR}/../include/gsl" )

set( SOURCES gsl-lite.t.cpp assert.t.cpp at.t.cpp byte.t.cpp issue.t.cpp not_null.t.cpp owner.t.cpp span.t.cpp string_span.t.cpp util.t.cpp )

set( GSL_CONFIG -Dgsl_CONFIG_CONTRACT_VIOLATION_THROWS )

set( HAS_STD_FLAGS  FALSE )
set( HAS_CPP98_FLAG FALSE )
set( HAS_CPP03_FLAG FALSE )
set( HAS_CPP11_FLAG FALSE )
set( HAS_CPP14_FLAG FALSE )
set( HAS_CPP17_FLAG FALSE )
set( HAS_CPP20_FLAG FALSE )

if( MSVC )
    set( HAS_STD_FLAGS TRUE )

    set( STD_OPT     -std: )
    set( OPTIONS     -W3 -EHsc )
    set( DEFINITIONS -D_SCL_SECURE_NO_WARNINGS -DNOMINMAX ${GSL_CONFIG} )

    if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.00 )
        set( HAS_CPP14_FLAG TRUE )
        set( HAS_CPPLATEST_FLAG TRUE )
    endif()
    if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.11 )
        set( HAS_CPP17_FLAG TRUE )
    endif()

elseif( CMAKE_CXX_COMPILER_ID MATCHES "GNU" OR
        CMAKE_CXX_COMPILER_ID MATCHES "Clang" )

    set( HAS_STD_FLAGS  TRUE )
    set( HAS_CPP98_FLAG TRUE )
    set( HAS_CPP03_FLAG TRUE )

    set( STD_OPT     -std= )
    set( OPTIONS     -Wall -Wno-missing-braces -fno-elide-constructors -Wconversion -Wsign-conversion -Wno-string-conversion ${GSL_CONFIG} )
    set( DEFINITIONS "" )

    # GNU: available -std flags depends on version
    if( CMAKE_CXX_COMPILER_ID MATCHES "GNU" )
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.8.0 )
            set( HAS_CPP11_FLAG TRUE )
        endif()
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9.2 )
            set( HAS_CPP14_FLAG TRUE )
        endif()
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 7.1.0 )
            set( HAS_CPP17_FLAG TRUE )
        endif()
    endif()

    # Clang: available -std flags depends on version
    if( CMAKE_CXX_COMPILER_ID MATCHES Clang )
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.3.0 )
            set( HAS_CPP11_FLAG TRUE )
        endif()
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.4.0 )
            set( HAS_CPP14_FLAG TRUE )
        endif()
        if( NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5.0.0 )
            set( HAS_CPP17_FLAG TRUE )
        endif()
    endif()

elseif( CMAKE_CXX_COMPILER_ID MATCHES "Intel" )
# as is
else()
# as is
endif()

# enable MS C++ Core Guidelines checker if MSVC:

function( enable_msvs_guideline_checker target )
    if( MSVC )
        set_target_properties( ${target} PROPERTIES
        VS_GLOBAL_EnableCppCoreCheck true
        VS_GLOBAL_CodeAnalysisRuleSet CppCoreCheckRules.ruleset
        VS_GLOBAL_RunCodeAnalysis true )
    endif()
endfunction()

# make target, compile for given standard if specified:

function( make_target target std )
    if( std )
        set( OPTIONAL_STD ${STD_OPT}${std} )
    endif()
    add_executable            ( ${target} ${SOURCES}  )
    target_compile_options    ( ${target} PRIVATE ${OPTIONS} ${OPTIONAL_STD} )
    target_compile_definitions( ${target} PRIVATE ${DEFINITIONS} )
    target_include_directories( ${target} PRIVATE ${INCDIRS} )
endfunction()

# add generic executable, unless -std flags can be specified:

if( NOT HAS_STD_FLAGS )
    make_target( gsl-lite.t "" )
else()
    # unconditionally add C++98 variant as MSVC has no option for it:
    if( HAS_CPP98_FLAG )
        make_target( gsl-lite-cpp98.t c++98 )
    else()
        make_target( gsl-lite-cpp98.t "" )
    endif()

    if( HAS_CPP03_FLAG )
        make_target( gsl-lite-cpp03.t c++03 )
    endif()

    if( HAS_CPP11_FLAG )
        make_target( gsl-lite-cpp11.t c++11 )
    endif()

    if( HAS_CPP14_FLAG )
        make_target( gsl-lite-cpp14.t c++14 )
    endif()

    if( HAS_CPP17_FLAG )
        make_target( gsl-lite-cpp17.t c++17 )
        enable_msvs_guideline_checker( gsl-lite-cpp17.t )
    endif()

    if( HAS_CPPLATEST_FLAG )
        make_target( gsl-lite-cpplatest.t c++latest )
    endif()
endif()

# configure unit tests via CTest:

enable_testing()

if( HAS_STD_FLAGS )
    # unconditionally add C++98 variant for MSVC:
    add_test(     NAME test-cpp98     COMMAND gsl-lite-cpp98.t )

    if( HAS_CPP03_FLAG )
        add_test( NAME test-cpp03     COMMAND gsl-lite-cpp03.t )
    endif()
    if( HAS_CPP11_FLAG )
        add_test( NAME test-cpp11     COMMAND gsl-lite-cpp11.t )
    endif()
    if( HAS_CPP14_FLAG )
        add_test( NAME test-cpp14     COMMAND gsl-lite-cpp14.t )
    endif()
    if( HAS_CPP17_FLAG )
        add_test( NAME test-cpp17     COMMAND gsl-lite-cpp17.t )
    endif()
    if( HAS_CPPLATEST_FLAG )
        add_test( NAME test-cpplatest COMMAND gsl-lite-cpplatest.t )
    endif()
else()
    add_test(     NAME test           COMMAND gsl-lite.t --pass )
    add_test(     NAME list_version   COMMAND gsl-lite.t --version )
    add_test(     NAME list_tags      COMMAND gsl-lite.t --list-tags )
    add_test(     NAME list_tests     COMMAND gsl-lite.t --list-tests )
endif()

# end of file

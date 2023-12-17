#
# tesseract unittest
#

# Prefetch googletest if it is not already initialized with `git submodule update --init`
include(FetchContent)
FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG main # as .gitmodules does not enforce releases tag we track the main branch
        SOURCE_DIR ${CMAKE_SOURCE_DIR}/unittest/third_party/googletest
)
FetchContent_Populate(googletest)
add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})

include(GoogleTest)

# core tests
set(TESSERACT_UNITTESTS
        apiexample
        cleanapi
        colpartition
        denorm
        fileio
        heap
        imagedata
        intsimdmatrix
        lang_model
        layout
        ligature_table
        linlsq
        list
        loadlang
        matrix
        networkio
        nthitem
        pagesegmode
        paragraphs
        progress
        qrsequence
        recodebeam
        rect
        resultiterator
        scanutils
        stats
        stridemap
        stringrenderer
        tablefind
        tablerecog
        tabvector
        tatweel
        tfile
)

# Legacy engine tests
if (NOT DISABLED_LEGACY_ENGINE)
    set(TESSERACT_UNITTESTS
            ${TESSERACT_UNITTESTS}
            equationdetect
            indexmapbidi
            intfeaturemap
            osd
            params_model
            shapetable
            textlineprojection
    )
endif ()

# training tests
if (BUILD_TRAINING_TOOLS)
    set(TESSERACT_UNITTESTS
            ${TESSERACT_UNITTESTS}
            baseapi
            baseapi_thread
            commandlineflags
            dawg
            lstm_recode
            lstm_squashed
            lstm
            lstmtrainer
            normstrngs
            pango_font_info
            unichar
            unicharcompress
            unicharset
            validate_grapheme
            validate_indic
            validate_khmer
            validate_myanmar
            validator
    )
    if (NOT DISABLED_LEGACY_ENGINE)
        set(TESSERACT_UNITTESTS
                ${TESSERACT_UNITTESTS}
                applybox
                bitvector
        )
    endif ()
endif ()

set(TESSBIN_DIR ${CMAKE_BINARY_DIR}/bin)
set(TESSDATA_DIR ${CMAKE_SOURCE_DIR}/../tessdata)
set(LANGDATA_DIR ${CMAKE_SOURCE_DIR}/../langdata_lstm)
set(TESTING_DIR ${CMAKE_SOURCE_DIR}/test/testing)
set(TESTDATA_DIR ${CMAKE_SOURCE_DIR}/test/testdata)

set(TESSERACT_UNITTEST_CXXFLAG
        -DTESSBIN_DIR="${TESSBIN_DIR}"
        -DTESSDATA_DIR="${TESSDATA_DIR}"
        -DLANGDATA_DIR="${LANGDATA_DIR}"
        -DTESTING_DIR="${TESTING_DIR}"
        -DTESTDATA_DIR="${TESTDATA_DIR}"
)

add_custom_target(build_tests)

foreach (unittest ${TESSERACT_UNITTESTS})
    add_executable(
            ${unittest}_test
            EXCLUDE_FROM_ALL
            ${unittest}_test.cc
            third_party/utf/rune.c
            util/utf8/unicodetext.cc
            util/utf8/unilib.cc
    )
    add_dependencies(${unittest}_test tesseract)
    target_compile_options(${unittest}_test PRIVATE ${TESSERACT_UNITTEST_CXXFLAG})
    target_include_directories(${unittest}_test PRIVATE ${CMAKE_SOURCE_DIR}/unittest)

    target_link_libraries(
            ${unittest}_test
            gtest
            gmock
            gtest_main
            libtesseract
    )

    if (BUILD_TRAINING_TOOLS)
        target_link_libraries(
                ${unittest}_test
                common_training
                unicharset_training
                pango_training
        )
    endif ()

    gtest_discover_tests(${unittest}_test)
    add_test(NAME ${unittest}_test COMMAND ${unittest}_test)
    add_dependencies(build_tests ${unittest}_test)
endforeach ()

add_custom_target(check COMMAND ctest)
add_dependencies(check build_tests)
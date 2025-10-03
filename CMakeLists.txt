cmake_minimum_required( VERSION 3.22 )

project( rnnoise LANGUAGES C CXX )

add_library( ${PROJECT_NAME} STATIC )

file ( STRINGS "model_version" RNNOISE_HASH )
set ( RNNOISE_MODEL rnnoise_data-${RNNOISE_HASH}.tar.gz )

include(FetchContent)

FetchContent_Declare( rnnoise_data
	URL https://media.xiph.org/rnnoise/models/${RNNOISE_MODEL}
	URL_HASH SHA256=${RNNOISE_HASH}
	DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)
FetchContent_MakeAvailable(rnnoise_data)

file( COPY ${rnnoise_data_SOURCE_DIR}/src/rnnoise_data_little.c
    DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/src)
	
file( RENAME ${CMAKE_CURRENT_SOURCE_DIR}/src/rnnoise_data_little.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rnnoise_data.c)
	
file( COPY ${rnnoise_data_SOURCE_DIR}/src/rnnoise_data_little.h
	DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/src)
	
file( RENAME ${CMAKE_CURRENT_SOURCE_DIR}/src/rnnoise_data_little.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/rnnoise_data.h)
	 
target_sources( rnnoise
    PRIVATE
        src/denoise.c
        src/rnn.c
        src/pitch.c
        src/kiss_fft.c
        src/celt_lpc.c
        src/nnet.c
        src/nnet_default.c
        src/parse_lpcnet_weights.c
        src/rnnoise_tables.c
        src/rnnoise_data.c
)

target_include_directories( rnnoise
    PUBLIC
        include
)

target_include_directories( rnnoise
    PRIVATE
        src
)

if( CMAKE_OSX_ARCHITECTURES )
  list( JOIN CMAKE_OSX_ARCHITECTURES "-" ARCH_NAME )
elseif( MSVC_CXX_ARCHITECTURE_ID )
  set( ARCH_NAME ${MSVC_CXX_ARCHITECTURE_ID} )
else()
  set( ARCH_NAME ${CMAKE_SYSTEM_PROCESSOR} )
endif()

STRING( TOLOWER "${ARCH_NAME}" ARCH_NAME )
set_target_properties( ${PROJECT_NAME} PROPERTIES
  PREFIX ""
  OUTPUT_NAME "lib_${ARCH_NAME}_${PROJECT_NAME}"
)
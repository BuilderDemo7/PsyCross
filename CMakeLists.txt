file(GLOB_RECURSE PSYCROSS_SRCS_C
	"*.c"
)

file(GLOB_RECURSE PSYCROSS_SRCS_CPP
	"*.cpp"
)

set_source_files_properties( ${PSYCROSS_SRCS_C} PROPERTIES LANGUAGE C)
set_source_files_properties( ${PSYCROSS_SRCS_CPP} PROPERTIES LANGUAGE CXX)

# C11 uses _Static_assert, but PsyCross headers use C++ style static_assert
# Add compatibility define for C compilation
set_source_files_properties( ${PSYCROSS_SRCS_C} PROPERTIES
    COMPILE_DEFINITIONS "static_assert=_Static_assert"
)
#set_source_files_properties( "src/gte/PsyX_GTE.cpp" PROPERTIES LANGUAGE C)

add_library(psycross_static STATIC ${PSYCROSS_SRCS_C} ${PSYCROSS_SRCS_CPP})
set_target_properties(psycross_static PROPERTIES OUTPUT_NAME "psycross")
target_include_directories(psycross_static PUBLIC "include" "include/psx")

find_package(SDL2 REQUIRED)
target_link_libraries(psycross_static ${SDL2_LIBRARIES})
target_include_directories(psycross_static PRIVATE ${SDL2_INCLUDE_DIRS})

find_package(OpenAL REQUIRED)
# find_package(OpenAL) exposes different handles depending on how OpenAL was
# found: the imported target OpenAL::OpenAL (openal-soft's CMake config, or
# CMake's FindOpenAL in 3.x), a bare `OpenAL` target (some MSYS2 configs), or
# just the OPENAL_LIBRARY / OPENAL_INCLUDE_DIR variables. Prefer the imported
# target; fall back so this links on both MinGW and Linux.
if(TARGET OpenAL::OpenAL)
    target_link_libraries(psycross_static OpenAL::OpenAL)
elseif(TARGET OpenAL)
    target_link_libraries(psycross_static OpenAL)
else()
    target_link_libraries(psycross_static ${OPENAL_LIBRARY})
    target_include_directories(psycross_static PRIVATE ${OPENAL_INCLUDE_DIR})
endif()
target_include_directories(psycross_static PRIVATE ${OpenAL_BINARY_DIR} ${OpenAL_SOURCE_DIR}/include)

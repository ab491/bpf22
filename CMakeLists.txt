cmake_minimum_required(VERSION 3.12)
project(BpfFilters)

find_package(LLVM REQUIRED CONFIG)

add_custom_target(filters ALL)

file(GLOB_RECURSE INPUT_FILES LIST_DIRECTORIES false src/*.c)

macro(add_some_kind_of_target target_name infile outfile)
    add_custom_target(${target_name} DEPENDS ${outfile})
    add_custom_command(
        TARGET ${target_name}
        COMMAND clang -O2 -emit-llvm -c ${infile} -o - | llc -march=bpf -filetype=obj -o ${outfile}
        WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
        COMMENT "Compiling ${infile}"
        VERBATIM 
    )
    
endmacro()


foreach(fil_path ${INPUT_FILES})
    get_filename_component(infilename ${fil_path} NAME)
    string(REGEX REPLACE ".c\$" ".o" outfilename ${infilename})
    set(infile_path ${PROJECT_SOURCE_DIR}/src/${infilename})

    add_some_kind_of_target(${outfilename} ${infile_path} ${outfilename})
    add_dependencies(filters ${outfilename})

    INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/${outfilename} DESTINATION share)
endforeach()

cmake_minimum_required(VERSION 3.5.1)

project(leg_beta)

message("PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}")
set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/..)
message("PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}")

message("PROJECT_BINARY_DIR ${PROJECT_BINARY_DIR}")
set(PROJECT_BINARY_DIR ${PROJECT_BINARY_DIR}/build_doc)
message("PROJECT_BINARY_DIR ${PROJECT_BINARY_DIR}")


#################
# Doxygen Macro #
#################

# Find "doxygen"
find_package(Doxygen)
if (NOT DOXYGEN_FOUND)
    message(FATAL_ERROR
        "Doxygen is needed to build the documentation. "
        "Please install it correctly")
endif()

# set the destination folder to be devel/share/[project_name]/doc/
set(DOXYGEN_DOXYFILE_IN ${PROJECT_SOURCE_DIR}/config_doc/doxygen/Doxyfile.in)
set(DOXYGEN_DOXYFILE ${PROJECT_BINARY_DIR}/docs/doxygen/Doxyfile)
set(DOXYGEN_OUTPUT ${PROJECT_BINARY_DIR}/docs/doxygen)

# Create the doxyfile in function of the current project.
# If the Doxyfile.in does not exists, the cmake step stops.
configure_file(${DOXYGEN_DOXYFILE_IN} ${DOXYGEN_DOXYFILE} @ONLY IMMEDIATE)

# the doxygen target is generated
add_custom_target (${PROJECT_NAME}_doxygen ALL
    COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_DOXYFILE}
    SOURCES ${DOXYGEN_DOXYFILE} # if the file change rebuild
    WORKING_DIRECTORY ${DOXYGEN_OUTPUT}
    COMMENT "Building doxygen documentation for ${PROJECT_NAME}")


##################
# Breathe APIDOC #
##################

# Find the breathe
find_program(BREATHE_APIDOC breathe-apidoc)
if(NOT BREATHE_APIDOC)
    message(FATAL_ERROR "breathe-apidoc not found!"
            "Please install using: pip3 install breathe")
endif()
set(BREATHE_IN      ${PROJECT_BINARY_DIR}/docs/doxygen/xml)
set(BREATHE_OUT     ${PROJECT_BINARY_DIR}/docs/sphinx/breathe)
set(BREATHE_OPTION  -g union,namespace,class,group,struct,file,interface)

file(MAKE_DIRECTORY ${BREATHE_OUT})
add_custom_target(
    ${PROJECT_NAME}_breathe_apidoc ALL
    # Generate the .rst files from the doxygen xml output
    ${BREATHE_APIDOC} -o ${BREATHE_OUT} ${BREATHE_IN} ${BREATHE_OPTION} 
    WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/docs/sphinx
    DEPENDS ${PROJECT_NAME}_doxygen
    COMMENT "Building breathe-apidoc for ${PROJECT_NAME}")



################
# SPHINX BUILD #
################

# Find the sphinx-apidoc executable.
find_program(SPHINX_BUILD sphinx-build)
if(NOT SPHINX_BUILD)
    message(FATAL_ERROR "sphinx-apidoc not found!"
            "Please install using: pip3 install sphinx")
endif()    

# Setup the parameters
set(SPHINX_OUTPUT ${PROJECT_BINARY_DIR}/docs/sphinx)
set(DOXYGEN_XML_OUTPUT ${PROJECT_BINARY_DIR}/docs/doxygen/xml)
set(SPHINX_IN      ${PROJECT_BINARY_DIR}/docs/sphinx)
set(SPHINX_OUT     ${PROJECT_SOURCE_DIR}/documentation)
set(SPHINX_OPTION  -Q) # quiet the sphinx output

# Generate the configuration files
set(SPHINX_CONFIG ${PROJECT_SOURCE_DIR}/config_doc/sphinx)

configure_file(${SPHINX_CONFIG}/conf.py.in                      ${SPHINX_OUTPUT}/conf.py                        @ONLY IMMEDIATE)
configure_file(${SPHINX_CONFIG}/doxygen_index_one_page.rst.in   ${SPHINX_OUTPUT}/doxygen_index_one_page.rst     @ONLY IMMEDIATE)
configure_file(${SPHINX_CONFIG}/doxygen_index.rst.in            ${SPHINX_OUTPUT}/doxygen_index.rst              @ONLY IMMEDIATE)
configure_file(${SPHINX_CONFIG}/index.rst.in                    ${SPHINX_OUTPUT}/index.rst                      @ONLY IMMEDIATE)

# Create the output
file(MAKE_DIRECTORY ${SPHINX_OUT})

add_custom_target(copy-runtime-files ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${PROJECT_SOURCE_DIR}/config_doc/sphinx  ${PROJECT_BINARY_DIR}/docs/sphinx
    DEPENDS ${MY_TARGET})

add_custom_target(
    ${PROJECT_NAME}_sphinx_html ALL
    ${SPHINX_BUILD} -M html ${SPHINX_IN} ${SPHINX_OUT} ${SPHINX_OPTION}
    WORKING_DIRECTORY ${SPHINX_OUTPUT}
    DEPENDS ${PROJECT_NAME}_breathe_apidoc copy-runtime-files
    COMMENT "Building sphinx-apidoc for ${PROJECT_NAME}")


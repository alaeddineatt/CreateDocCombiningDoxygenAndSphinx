cmake_minimum_required(VERSION 3.5.1)

project(test_ws)

#################
# Doxygen Macro #
#################

macro(_BUILD_DOXYGEN)

    # Find "doxygen"
    find_package(Doxygen)
    if (NOT DOXYGEN_FOUND)
        message(FATAL_ERROR
            "Doxygen is needed to build the documentation. "
            "Please install it correctly")
    endif()

    message(${PROJECT_SOURCE_DIR})
    message(${PROJECT_BINARY_DIR})

    # set the destination folder to be devel/share/[project_name]/doc/
    set(DOXYGEN_DOXYFILE_IN ${PROJECT_SOURCE_DIR}/doc_config_files/doxygen/Doxyfile.in)
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

endmacro(_BUILD_DOXYGEN)


##################
# Breathe APIDOC #
##################

macro(_BUILD_BREATHE_APIDOC)

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

endmacro(_BUILD_BREATHE_APIDOC)


##################
# SPHINX APIDOC #
##################

macro(_BUILD_SPHINX_API_DOC)

    # Find the sphinx-apidoc executable.
    find_program(SPHINX_APIDOC sphinx-apidoc)
    if(NOT SPHINX_APIDOC)
        message(FATAL_ERROR "sphinx-apidoc not found!"
                "Please install using: pip3 install sphinx")
    endif()
    set(SPHINX_IN      ${PROJECT_SOURCE_DIR}/src/py_pubsub/py_pubsub)
    set(SPHINX_OUT     ${PROJECT_BINARY_DIR}/docs/sphinx)

    # Create the output
    file(MAKE_DIRECTORY ${SPHINX_OUT})

    message(${SPHINX_APIDOC})

    add_custom_target(
        ${PROJECT_NAME}_sphinx_apidoc ALL
	    ${SPHINX_APIDOC} -o ${SPHINX_OUT} ${SPHINX_IN}
        WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/docs/sphinx
        DEPENDS ${PROJECT_NAME}_breathe_apidoc
        COMMENT "Building sphinx-apidoc for ${PROJECT_NAME}")

endmacro(_BUILD_SPHINX_API_DOC)

################
# SPHINX BUILD #
################

macro(_BUILD_SPHINX_BUILD)
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
    set(SPHINX_OUT     ${SPHINX_IN})
    set(SPHINX_OPTION  -Q) # quiet the sphinx output

    # Generate the configuration files

    configure_file(${PROJECT_SOURCE_DIR}/doc_config_files/sphinx/conf.py.in
                   ${SPHINX_OUTPUT}/conf.py @ONLY IMMEDIATE)
    configure_file(${PROJECT_SOURCE_DIR}/doc_config_files/sphinx/doxygen_index_one_page.rst.in
                   ${SPHINX_OUTPUT}/doxygen_index_one_page.rst @ONLY IMMEDIATE)
    configure_file(${PROJECT_SOURCE_DIR}/doc_config_files/sphinx/doxygen_index.rst.in
                   ${SPHINX_OUTPUT}/doxygen_index.rst @ONLY IMMEDIATE)
    configure_file(${PROJECT_SOURCE_DIR}/doc_config_files/sphinx/index.rst.in
                   ${SPHINX_OUTPUT}/index.rst @ONLY IMMEDIATE)

    # Create a symlink to the doc fodler conatining the Markdown files.
    add_custom_target(
        ${PROJECT_NAME}_doc_symlink ALL
        ${CMAKE_COMMAND} -E create_symlink ${PROJECT_SOURCE_DIR}/doc ${SPHINX_OUTPUT}/doc
        WORKING_DIRECTORY ${SPHINX_OUTPUT}
        COMMENT "make a symlink ${src} -> ${dest}")

    # Create the output
    file(MAKE_DIRECTORY ${SPHINX_OUT})
    add_custom_target(
        ${PROJECT_NAME}_sphinx_html ALL
	    ${SPHINX_BUILD} -M html ${SPHINX_IN} ${SPHINX_OUT} ${SPHINX_OPTION}
        WORKING_DIRECTORY ${SPHINX_OUTPUT}
        # DEPENDS ${PROJECT_NAME}_sphinx_apidoc ${PROJECT_NAME}_doc_symlink
        COMMENT "Building sphinx-apidoc for ${PROJECT_NAME}")

endmacro(_BUILD_SPHINX_BUILD)

# # Copy the config files
# cp -r doc_config_files/sphinx/* $(DOC_BUILD_DIR)/sphinx

# #copy the markdown doc files
# cd $(DOC_BUILD_DIR)/sphinx && ln -sf ../../../doc && cd -

# # Generate the final layout.
# @$(SPHINX_BUILD) -M $@ "$(SPHINX_BUILD_IN)" "$(SPHINX_BUILD_OUT)" $(SPHINXOPTS) $(O) $(SPHINX_QUIET)

##########################
# building documentation #
##########################

# First we need to build the doxygen xml files.
_build_doxygen()
_build_breathe_apidoc()
# _build_sphinx_api_doc()
_build_sphinx_build()


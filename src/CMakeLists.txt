# Copyright (C) 2017-2018 Jonathan Müller <jonathanmueller.dev@gmail.com>
# This file is subject to the license terms in the LICENSE file
# found in the top-level directory of this distribution.

set(detail_header
        ../include/cppast/detail/assert.hpp
        ../include/cppast/detail/intrusive_list.hpp)
set(header
    ../include/cppast/code_generator.hpp
    ../include/cppast/compile_config.hpp
    ../include/cppast/cpp_alias_template.hpp
    ../include/cppast/cpp_array_type.hpp
    ../include/cppast/cpp_attribute.hpp
    ../include/cppast/cpp_class.hpp
    ../include/cppast/cpp_class_template.hpp
    ../include/cppast/cpp_decltype_type.hpp
    ../include/cppast/cpp_entity.hpp
    ../include/cppast/cpp_entity_container.hpp
    ../include/cppast/cpp_entity_index.hpp
    ../include/cppast/cpp_entity_kind.hpp
    ../include/cppast/cpp_entity_ref.hpp
    ../include/cppast/cpp_enum.hpp
    ../include/cppast/cpp_expression.hpp
    ../include/cppast/cpp_file.hpp
    ../include/cppast/cpp_forward_declarable.hpp
    ../include/cppast/cpp_friend.hpp
    ../include/cppast/cpp_function.hpp
    ../include/cppast/cpp_function_template.hpp
    ../include/cppast/cpp_function_type.hpp
    ../include/cppast/cpp_language_linkage.hpp
    ../include/cppast/cpp_member_function.hpp
    ../include/cppast/cpp_member_variable.hpp
    ../include/cppast/cpp_namespace.hpp
    ../include/cppast/cpp_preprocessor.hpp
    ../include/cppast/cpp_static_assert.hpp
    ../include/cppast/cpp_storage_class_specifiers.hpp
    ../include/cppast/cpp_template.hpp
    ../include/cppast/cpp_template_parameter.hpp
    ../include/cppast/cpp_token.hpp
    ../include/cppast/cpp_type.hpp
    ../include/cppast/cpp_type_alias.hpp
    ../include/cppast/cpp_variable.hpp
    ../include/cppast/cpp_variable_base.hpp
    ../include/cppast/cpp_variable_template.hpp
    ../include/cppast/diagnostic.hpp
    ../include/cppast/diagnostic_logger.hpp
    ../include/cppast/libclang_parser.hpp
    ../include/cppast/parser.hpp
    ../include/cppast/visitor.hpp)
set(source
        code_generator.cpp
        cpp_alias_template.cpp
        cpp_attribute.cpp
        cpp_class.cpp
        cpp_class_template.cpp
        cpp_entity.cpp
        cpp_entity_index.cpp
        cpp_entity_kind.cpp
        cpp_enum.cpp
        cpp_expression.cpp
        cpp_file.cpp
        cpp_forward_declarable.cpp
        cpp_friend.cpp
        cpp_function.cpp
        cpp_function_template.cpp
        cpp_language_linkage.cpp
        cpp_member_function.cpp
        cpp_member_variable.cpp
        cpp_namespace.cpp
        cpp_preprocessor.cpp
        cpp_static_assert.cpp
        cpp_template_parameter.cpp
        cpp_token.cpp
        cpp_type.cpp
        cpp_type_alias.cpp
        cpp_variable.cpp
        cpp_variable_template.cpp
        diagnostic_logger.cpp
        visitor.cpp)
set(libclang_source
        libclang/class_parser.cpp
        libclang/cxtokenizer.cpp
        libclang/cxtokenizer.hpp
        libclang/debug_helper.cpp
        libclang/debug_helper.hpp
        libclang/enum_parser.cpp
        libclang/expression_parser.cpp
        libclang/friend_parser.cpp
        libclang/function_parser.cpp
        libclang/language_linkage_parser.cpp
        libclang/libclang_parser.cpp
        libclang/libclang_visitor.hpp
        libclang/namespace_parser.cpp
        libclang/parse_error.hpp
        libclang/parse_functions.cpp
        libclang/parse_functions.hpp
        libclang/preprocessor.cpp
        libclang/preprocessor.hpp
        libclang/raii_wrapper.hpp
        libclang/template_parser.cpp
        libclang/type_parser.cpp
        libclang/variable_parser.cpp)

add_library(cppast ${detail_header} ${header} ${source} ${libclang_source})
target_compile_features(cppast PUBLIC cxx_std_11)
target_include_directories(cppast PRIVATE ../include SYSTEM INTERFACE
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
target_link_libraries(cppast PUBLIC type_safe tiny-process-library::tiny-process-library libclang)
target_compile_definitions(cppast PUBLIC
                                CPPAST_CLANG_BINARY="${LLVM_ROOT}/bin/clang"
                                CPPAST_VERSION_MINOR="${cppast_VERSION_MINOR}"
                                CPPAST_VERSION_MAJOR="${cppast_VERSION_MAJOR}"
                                CPPAST_VERSION_STRING="${cppast_VERSION}")
target_compile_options(cppast PRIVATE
                           # clang/GCC warnings
                           $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:GNU>>:
                           -pedantic-errors -Werror -Wall -Wextra -Wconversion -Wsign-conversion>
                           # disable noexcept type warning on GCC
                           $<$<CXX_COMPILER_ID:GNU>: -Wno-noexcept-type>
                           # MSVC warnings
                           $<$<CXX_COMPILER_ID:MSVC>:
                           /WX /W4>)

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

install(TARGETS cppast
 EXPORT "${TARGETS_EXPORT_NAME}"
 LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
 ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

include(CMakePackageConfigHelpers)

# Use:
# * PROJECT_VERSION
write_basic_package_version_file(
   "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use:
# * TARGETS_EXPORT_NAME
# * PROJECT_NAME
configure_package_config_file(
   "${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in"
   "${project_config}"
   INSTALL_DESTINATION "${config_install_dir}"
)

install(
   FILES "${project_config}" "${version_config}"
   DESTINATION "${config_install_dir}"
)

install(
   EXPORT "${TARGETS_EXPORT_NAME}"
   NAMESPACE "${namespace}"
   DESTINATION "${config_install_dir}"
)

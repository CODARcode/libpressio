cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
find_package(Python3 COMPONENTS Interpreter Development NumPy)
find_package(SWIG)
if(SWIG_FOUND AND PYTHON3_FOUND)
  include(UseSWIG)


  set(CMAKE_SWIG_FLAGS "-doxygen")
  function(build_pressio_swig_module)
    get_filename_component(filelibname ${ARGV0} NAME_WE)
    set_property(SOURCE ${ARGV0} PROPERTY CPLUSPLUS ON)
    set_property(SOURCE ${ARGV0} PROPERTY USE_TARGET_INCLUDE_DIRECTORIES ON)
    set_property(SOURCE ${ARGV0}.i PROPERTY SWIG_MODULE_NAME ${filelibname})
    swig_add_library(
      ${filelibname}
      LANGUAGE python
      SOURCES ${ARGV}
      )
    set_property(TARGET ${filelibname} PROPERTY SWIG_COMPILE_DEFINITIONS "SWIGWORDSIZE64")
    target_link_libraries(${filelibname} PUBLIC SZ libpressio Python3::Python Python3::NumPy)
    target_include_directories(${filelibname} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
    if(LIBPRESSIO_HAS_MPI4PY)
      execute_process(
        COMMAND ${Python3_EXECUTABLE} -c "import mpi4py; print(mpi4py.get_include())" 
        OUTPUT_VARIABLE MPI4PY_SWIG_DIR 
        RESULT_VARIABLE MPI4PY_FOUND 
        OUTPUT_STRIP_TRAILING_WHITESPACE
      )
      if(NOT ${MPI4PY_FOUND} EQUAL "0")
        message(FATAL_ERROR "Could not locate mpi4py")
      endif()
      target_include_directories(${filelibname} PRIVATE ${MPI4PY_SWIG_DIR})
    endif()
    get_property(swig_generated_module TARGET ${filelibname} PROPERTY SWIG_SUPPORT_FILES)

    install(TARGETS ${filelibname} DESTINATION ${Python3_SITELIB})
    install(FILES ${swig_generated_module} DESTINATION ${Python3_SITELIB})

  endfunction()


  build_pressio_swig_module(pressio.i)
  install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/libpressio.py" DESTINATION ${Python3_SITELIB})
endif()


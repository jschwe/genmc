cmake_minimum_required(VERSION 3.21)
project(genmc VERSION "0.10" LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#find_package(LLVM 16 REQUIRED)
#message(STATUS "LLVM version ${LLVM_VERSION}, include dir: ${LLVM_INCLUDE_DIRS}")
set(LLVM_INCLUDE_DIRS "/usr/include/llvm15")
set(CMAKE_INCLUDE_DIRECTORIES_BEFORE ON)
include_directories("${LLVM_INCLUDE_DIRS}")

# add_compile_options(-Wall -Werror )
include(CheckIncludeFileCXX)


function(check_header "header_name" out_cache_var_name)
    string(MAKE_C_IDENTIFIER "${header_name}" header_id)
    set(cache_var_name "HAVE_${header_id}")
    message( STATUS "var: ${cache_var_name}")
    if(NOT DEFINED CACHE{${cache_var_name})
        set(CMAKE_REQUIRED_INCLUDES "${LLVM_INCLUDE_DIRS}")
        CHECK_INCLUDE_FILE_CXX("${header_name}" "${cache_var_name}")
    endif()
    set("${out_cache_var_name}" "${cache_var_name}" PARENT_SCOPE)
endfunction()

set(genmc_config_h "#pragma once\n")

set(required_headers
    "stdlib.h"
    # "llvm/ExecutionEngine/ExecutionEngine.h"
    "llvm/IR/LegacyPassManager.h"
)

foreach(header ${required_headers})
    check_header("${header}" has_header_var)
    string(APPEND genmc_config_h
        "#cmakedefine01 ${has_header_var}\n"
    )
endforeach()

string(APPEND genmc_config_h
       "#define HAVE_LLVM_EXECUTIONENGINE_EXECUTIONENGINE_H\n"
)

#file(CONFIGURE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/include/config.h"
#    CONTENT "${genmc_config_h}"
#)
include_directories("${CMAKE_CURRENT_BINARY_DIR}/include")

find_library(HAVE_LIB_DL NAMES "dl" "dlopen" REQUIRED)
find_library(HAVE_LIB_FFI NAMES "ffi" "ffi_call" REQUIRED)
find_library(HAVE_LIB_HWLOC NAMES "hwloc" "hwloc_topology_init")


# todo: port checks for alternative headers.


add_library(genmc STATIC
            src/ASize.cpp src/ASize.hpp
  src/BisimilarityCheckerPass.cpp src/BisimilarityCheckerPass.hpp
  src/Bitmask.hpp
  src/BoundDecider.cpp src/BoundDecider.hpp
  src/CallInfoCollectionPass.cpp src/CallInfoCollectionPass.hpp
  src/CodeCondenserPass.cpp src/CodeCondenserPass.hpp
  src/ContextBoundDecider.cpp src/ContextBoundDecider.hpp
  src/CallInstWrapper.hpp
  src/ConfirmationAnnotationPass.cpp src/ConfirmationAnnotationPass.hpp
  src/Config.cpp src/Config.hpp
  src/DeclareInternalsPass.cpp src/DeclareInternalsPass.hpp
  src/DefineLibcFunsPass.cpp src/DefineLibcFunsPass.hpp
  src/DepExecutionGraph.cpp src/DepExecutionGraph.hpp
  src/DepInfo.cpp src/DepInfo.hpp
  src/DepTracker.hpp
  src/DepView.cpp src/DepView.hpp
  src/DriverFactory.hpp
  src/EliminateAnnotationsPass.cpp src/EliminateAnnotationsPass.hpp
  src/EliminateCASPHIsPass.cpp src/EliminateCASPHIsPass.hpp
  src/EliminateCastsPass.cpp src/EliminateCastsPass.hpp
  src/EliminateRedundantInstPass.cpp src/EliminateRedundantInstPass.hpp
  src/EliminateUnusedCodePass.cpp src/EliminateUnusedCodePass.hpp
  src/Error.hpp
  src/EscapeCheckerPass.cpp src/EscapeCheckerPass.hpp
  src/Event.cpp src/Event.hpp
  src/EventAttr.hpp
  src/EventLabel.cpp src/EventLabel.hpp
  src/Execution.cpp
  src/ExecutionGraph.cpp src/ExecutionGraph.hpp
  src/ExternalFunctions.cpp
  src/FunctionInlinerPass.cpp src/FunctionInlinerPass.hpp
  src/GenMCDriver.cpp src/GenMCDriver.hpp
  src/GraphIterators.hpp
  src/IMMDriver.cpp src/IMMDriver.hpp
  src/InstAnnotator.cpp src/InstAnnotator.hpp
  src/Interpreter.cpp
  src/InterpreterEnumAPI.cpp src/InterpreterEnumAPI.hpp
  src/IntrinsicLoweringPass.cpp src/IntrinsicLoweringPass.hpp
  src/LabelVisitor.hpp
  src/LoadAnnotationPass.cpp src/LoadAnnotationPass.hpp
  src/LocalSimplifyCFGPass.cpp src/LocalSimplifyCFGPass.hpp
  src/Logger.hpp
  src/LLVMModule.cpp src/LLVMModule.hpp
  src/LLVMUtils.cpp src/LLVMUtils.hpp
  src/LoopUnrollPass.cpp src/LoopUnrollPass.hpp
  src/LoopJumpThreadingPass.cpp src/LoopJumpThreadingPass.hpp
  src/Matrix2D.hpp # src/Matrix2D.tcc
  src/MaximalIterator.hpp
  src/MDataCollectionPass.hpp src/MDataCollectionPass.cpp
  src/MemAccess.hpp
  src/MMDetectorPass.cpp src/MMDetectorPass.hpp
  src/ModuleID.hpp
  src/ModuleInfo.cpp src/ModuleInfo.hpp
  src/NameInfo.cpp src/NameInfo.hpp
  src/Parser.cpp src/Parser.hpp
  src/PromoteMemIntrinsicPass.cpp src/PromoteMemIntrinsicPass.hpp
  src/PropagateAssumesPass.cpp src/PropagateAssumesPass.hpp
  src/RADriver.cpp src/RADriver.hpp
  src/RC11Driver.cpp src/RC11Driver.hpp
  src/Revisit.cpp src/Revisit.hpp
  src/RoundBoundDecider.cpp src/RoundBoundDecider.hpp
  src/SAddr.cpp src/SAddr.hpp
  src/SAddrAllocator.hpp
  src/SCDriver.cpp src/SCDriver.hpp
  src/SExpr.tcc src/SExpr.hpp
  src/SExprVisitor.tcc src/SExprVisitor.hpp
  src/Stamp.cpp src/Stamp.hpp
  src/SVal.cpp src/SVal.hpp
  src/SpinAssumePass.cpp src/SpinAssumePass.hpp
  src/ThreadInfo.hpp
  src/ThreadPinner.cpp src/ThreadPinner.hpp
  src/ThreadPool.cpp src/ThreadPool.hpp
  src/TSODriver.cpp src/TSODriver.hpp
  src/WorkSet.hpp src/WorkSet.cpp
  src/value_ptr.hpp
  src/Verbosity.hpp src/Verbosity.cpp
  src/VectorClock.hpp src/VectorClock.cpp
  src/VerificationError.hpp src/VerificationError.cpp
  src/View.hpp src/View.cpp
  src/VSet.hpp src/VSet.tcc

)

# Todo: Don't hardcode this.
find_library(llvm_lib LLVM PATHS /usr/lib64/llvm15/lib NO_DEFAULT_PATH)

# Note: I don't like hardcoding the location, but this is how it was in the autotools version.
add_compile_definitions("INCLUDE_DIR=\"${CMAKE_INSTALL_PREFIX}/${CMAKE_PROJECT_NAME}/include\""
                        "SRC_INCLUDE_DIR=\"${CMAKE_SOURCE_DIR}/include\"")
add_executable(genmc-cli src/main.cpp)
target_link_libraries(genmc-cli PRIVATE genmc)
target_link_libraries(genmc-cli PRIVATE ffi dl "${llvm_lib}")

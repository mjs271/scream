include (ScreamUtils)

# Create the test
SET (TEST_LABELS "mam4;physics;driver")
set (NEED_LIBS mam scream_control scream_share diagnostics)
CreateUnitTest(mam4_nucleation_standalone "mam4_nucleation_standalone.cpp" "${NEED_LIBS}" LABELS ${TEST_LABELS}
  MPI_RANKS ${TEST_RANK_START} ${TEST_RANK_END}
  PROPERTIES FIXTURES_SETUP mam4_generate_output_nc_files
)

# Set AD configurable options
set (ATM_TIME_STEP 1800)
SetVarDependingOnTestSize(NUM_STEPS 2 5 48)  # 1h 2.5h 24h
set (RUN_T0 2021-10-12-45000)

## Copy (and configure) yaml files needed by tests
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/input.yaml
               ${CMAKE_CURRENT_BINARY_DIR}/input.yaml)
configure_file(mam4_nucleation_output.yaml mam4_nucleation_output.yaml)

# Ensure test input files are present in the data dir
GetInputFile(scream/init/${EAMxx_tests_IC_FILE_72lev})

## Finally compare all MPI rank output files against the single rank output as a baseline, using CPRNC
## Only if running with 2+ ranks configurations
# This test requires CPRNC
#if (TEST_RANK_END GREATER TEST_RANK_START)
#  include (BuildCprnc)
#  BuildCprnc()
#  SET (BASE_TEST_NAME "mam4_nucleation_standalone")
#  math (EXPR CMP_RANK_START ${TEST_RANK_START}+1)
#  foreach (MPI_RANKS RANGE ${CMP_RANK_START} ${TEST_RANK_END})
#
#    set (SRC_FILE "${BASE_TEST_NAME}_standalone_output.INSTANT.nsteps_x1.np${MPI_RANKS}.${RUN_T0}.nc")
#    set (TGT_FILE "${BASE_TEST_NAME}_standalone_output.INSTANT.nsteps_x1.np${TEST_RANK_START}.${RUN_T0}.nc")
#    set (TEST_NAME "${BASE_TEST_NAME}_np${TEST_RANK_START}_vs_np${MPI_RANKS}_bfb")
#    add_test (NAME ${TEST_NAME}
#              COMMAND cmake -P ${CMAKE_BINARY_DIR}/bin/CprncTest.cmake ${SRC_FILE} ${TGT_FILE}
#              WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
#    set_tests_properties(${TEST_NAME} PROPERTIES LABELS "${TEST_LABELS}"
#              RESOURCE_LOCK ${BASE_TEST_NAME}
#              FIXTURES_REQUIRED mam4_generate_output_nc_files)
#  endforeach()
#endif()
#
## Check tendency calculation
#foreach (NRANKS RANGE ${TEST_RANK_START} ${TEST_RANK_END})
#  set (script ${SCREAM_BASE_DIR}/scripts/check-tendencies)
#  set (fname mam4_nucleation_standalone_output.INSTANT.nsteps_x1.np${NRANKS}.${RUN_T0}.nc)
#  set (tname mam4_nucleation_tend_check_np${NRANKS})
#  add_test (NAME ${tname}
#    COMMAND ${script} -f ${fname} -v qc T_mid  #FIXME: Add mam4 tendencies -t p3_qc_tend p3_T_mid_tend
#    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
#  set_tests_properties (${tname} PROPERTIES
#    FIXTURES_REQUIRED mam4_generate_output_nc_files
#    LABELS "${TEST_LABELS}")
#endforeach()

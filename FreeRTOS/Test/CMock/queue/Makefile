# Change according to what your  unit test directory is.
# For example if testing queue.c your directory should be called queue
# and the project name should be queue
# if testing list.c your directory should be called list
# and the project name should be list
PROJECT          :=   queue

# List the dependency files you wish to mock
MOCK_FILES_FP    :=  $(KERNEL_DIR)/include/task.h
MOCK_FILES_FP    +=  $(KERNEL_DIR)/include/list.h

# List the options the compilation would need
CPPFLAGS         +=   -DportUSING_MPU_WRAPPERS=0

# Try not to edit beyond this line
MOCK_FILES       :=   $(notdir $(MOCK_FILES_FP))
MOCK_OBJ         :=   $(addprefix mock_,$(MOCK_FILES:.h=.o))
MOCK_SRC         :=   $(addprefix mock_,$(MOCK_FILES:.h=.c))
EXEC             :=   $(PROJECT)_utest
PROJECT_DIR      :=   $(abspath .)
SCRATCH_DIR      :=   $(GENERATED_DIR)/$(PROJECT)
PROJ_LIB_DIR     :=   $(SCRATCH_DIR)/lib
MOCK_OBJ_LIST    :=   $(addprefix $(PROJ_LIB_DIR)/,$(MOCK_OBJ))
MOCKS_DIR        :=   $(SCRATCH_DIR)/mocks
MOCK_SRC_LIST    :=   $(addprefix $(MOCKS_DIR)/,$(MOCK_SRC))
CFLAGS           +=   -I$(MOCKS_DIR)
COVERAGE_OPTS    := -fprofile-arcs -ftest-coverage -fprofile-generate

$(MOCKS_DIR)/mock_%.c : directories Makefile
	cd $(SCRATCH_DIR) && \
	    ruby $(CMOCK_EXEC_DIR)/cmock.rb -o$(PROJECT_DIR)/$(PROJECT).yml \
	    $(MOCK_FILES_FP)

$(PROJ_LIB_DIR)/mock_%.o : $(MOCKS_DIR)/mock_%.c
	$(CC) -c $< -fPIC $(CFLAGS) -o $@

$(BIN_DIR)/$(EXEC) : $(SCRATCH_DIR)/test_runner.o             \
		    $(SCRATCH_DIR)/$(PROJECT).o               \
		    $(SCRATCH_DIR)/$(PROJECT)_utest.o         \
		    $(MOCK_OBJ_LIST) | Makefile
	$(CC) $+ $(LDFLAGS) -o $@
 
$(SCRATCH_DIR)/test_runner.o : $(SCRATCH_DIR)/test_runner.c
	$(CC) -c $< $(CPPFLAGS)  $(CFLAGS) -o $@

$(SCRATCH_DIR)/$(PROJECT)_utest.o : $(PROJECT_DIR)/$(PROJECT)_utest.c
	$(CC) -c $< $(CPPFLAGS)  $(CFLAGS) -o $@

$(SCRATCH_DIR)/$(PROJECT).o : $(KERNEL_DIR)/$(PROJECT).c 
	$(CC) -c $< $(CPPFLAGS)  $(CFLAGS) $(COVERAGE_OPTS) -o $@

$(SCRATCH_DIR)/test_runner.c : Makefile $(MOCK_OBJ_LIST)
	ruby $(UNITY_BIN_DIR)/generate_test_runner.rb $(EXEC).c   \
		$(PROJECT_DIR)/$(PROJECT).yml $@

.PHONY: directories
directories :
	-mkdir -p $(SCRATCH_DIR)
	-mkdir -p $(MOCKS_DIR) 
	-mkdir -p $(PROJ_LIB_DIR) 

# prevent deletion by chain of implicit rules
NO_DELETE: $(MOCK_SRC_LIST)

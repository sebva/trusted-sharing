CC             := gcc
CXX            := g++-5
SHELL          := /bin/bash
SGX            := 0

SrcDir         := src
SGX_COMMONDIR  := $(realpath ../../sgx_common)
PBCRootDir     := $(realpath ../pbc-0.5.14)
GMPRootDir     := $(realpath ../gmp-6.1.2)

SgxToken       := sgx
NativeToken    := ntv

SharedCompFlags := -m64
CFlags          := -std=c11
CXXFlags        := -std=c++11
App_CFlags      := $(SharedCompFlags) $(CFlags)
Enclave         := grpshr
TARGET          := $(Enclave)
Untrusted_Objs  := $(TARGET).o
EnclaveName     := enclave_$(Enclave)
ExternalLibs    := pbc gmp 

IBBEDir     := ../pbc_ibbe
IBBEC       := $(notdir $(wildcard $(IBBEDir)/sgx_*.c))
IBBECXX     := $(notdir $(wildcard $(IBBEDir)/sgx_*.cpp))

ifeq ($(SGX),1)
BuildToken  := $(SgxToken)
else
BuildToken  := $(NativeToken)
endif

GMPDir          := $(GMPRootDir)/build-$(BuildToken)
PBCDir          := $(PBCRootDir)/build-$(BuildToken)
GMPLib          := $(GMPDir)/.libs/libgmp.a
PBCLib          := $(PBCDir)/.libs/libpbc.a
ExtLibs         := $(PBCLib) $(GMPLib)
LibsConfigFlags := --with-pic=yes --enable-shared=no

BinDir  := bin-$(BuildToken)
ObjDir  := obj-$(BuildToken)

ifeq ($(SGX),1)
IBBEC                := $(IBBEC:%.c=%_t.o)
IBBECXX              := $(IBBECXX:%.cpp=%_t.o)
IBBEObjs             := $(IBBEC) $(IBBECXX)
CommonEnclObjs       := libc_proxy_t.o file_mock_t.o sdk_deprecated_t.o sgx_cryptoall_t.o
TRUSTEDOBJS          := $(EnclaveName)_t.o ecalls_$(Enclave)_t.o \
                        $(CommonEnclObjs) $(IBBEObjs)
SGX_SDK              := /opt/intel/sgxsdk
SGX_LIBRARY_PATH     := $(SGX_SDK)/lib64
SGX_ENCLAVE_SIGNER   := $(SGX_SDK)/bin/x64/sgx_sign
SGX_EDGER8R          := $(SGX_SDK)/bin/x64/sgx_edger8r
Trts_Library_Name    := sgx_trts
Service_Library_Name := sgx_tservice
App_Lib_Dirs         := $(SGX_LIBRARY_PATH) 
App_Libs             := sgx_uae_service sgx_urts 
SharedCompFlags      += -DENABLE_SGX
IncludeDirs          += $(SGX_SDK)/include
Untrusted_Objs       += ocalls.o
Untrusted_Objs       := $(filter-out $(TRUSTEDOBJS), $(Untrusted_Objs)) \
	                    $(EnclaveName)_u.o
CommonNatvObjs       := sgx_initenclave sgx_errlist
SgxCommonObjs        := $(addsuffix _u.o,$(addprefix $(ObjDir)/, $(CommonNatvObjs)))
Enclave_Lib_Dirs     := $(SGX_LIBRARY_PATH) $(dir $(ExtLibs))
Enclave_Libs         := 
Enclave_SDK_Libs     := -Wl,--whole-archive \
                            -l$(Trts_Library_Name) \
                        -Wl,--no-whole-archive \
                        -Wl,--start-group \
                             $(addprefix -l, sgx_tstdc sgx_tstdcxx sgx_tcrypto \
                                             $(Service_Library_Name) ) \
                        -Wl,--end-group
Enclave_Link_Flags   := $(addprefix -L, $(Enclave_Lib_Dirs)) \
                        $(addprefix -l, $(Enclave_Libs)) \
                        $(Enclave_SDK_Libs)
Trusted_Link_Flags := -Wl,--no-undefined -nostdlib \
	-nodefaultlibs -nostartfiles $(Enclave_Link_Flags)  \
	-Wl,-Bstatic -Wl,-Bsymbolic -Wl,--no-undefined \
	-Wl,-pie,-eenclave_entry -Wl,--export-dynamic  \
	-Wl,--defsym,__ImageBase=0 \
	-Wl,--version-script=$(SrcDir)/$(EnclaveName).lds \
    $(addprefix -l, $(ExternalLibs) sgx_tstdc)
SharedEnclFlags    += $(SharedCompFlags) -nostdinc -fvisibility=hidden \
	                  -fpie -fstack-protector -fno-builtin-printf
Enclave_C_Flags    := $(SharedEnclFlags) $(CFlags) \
                      -Wno-implicit-function-declaration 
Enclave_CXXFlags  := $(SharedEnclFlags) $(CXXFlags) -nostdinc++
Enclave_IncDirs    := $(SGX_SDK)/include/stlport $(SrcDir) $(SGX_SDK)/include \
                     $(SGX_SDK)/include/tlibc $(SGX_COMMONDIR) \
                     $(PBCRootDir)/include $(GMPDir)
Enclave_Include    := $(addprefix -I, $(Enclave_IncDirs) $(IBBEDir))
App_Cpp_Objs       := $(SgxCommonObjs)
Trusted_Objects    := $(addprefix $(ObjDir)/,$(TRUSTEDOBJS))
LibsConfigFlags    += --enable-sgx
ObjSuffix          := t
CompString         := Encl
CompCFlags         := ${Enclave_C_Flags}
CompCXXFlags       := ${Enclave_CXXFlags}
all: $(BinDir)/$(TARGET) $(BinDir)/$(TARGET).signed.so
$(addprefix $(ObjDir)/,server_protocol.o $(TARGET).o): $(SrcDir)/$(EnclaveName)_u.c
$(ObjDir)/query_manager.o : $(SrcDir)/$(EnclaveName)_t.c
else
App_Lib_Dirs   := $(dir $(ExtLibs))
App_Libs       := $(ExternalLibs)
SgxCommonObjs  := $(addprefix $(ObjDir)/, sgx_cryptoall_u.o)
IBBEC          := $(IBBEC:%.c=%_u.o)
IBBECXX        := $(IBBECXX:%.cpp=%_u.o)
IBBEObjs    := $(IBBEC) $(IBBECXX)
Untrusted_Objs += ecalls_$(Enclave).o $(IBBEObjs)
IncludeDirs    += $(IBBEDir)
LDFLAGS        := -lcrypto -lcrypto++
ObjSuffix      := u
CompString     := Natv
CompCFlags     := ${App_CFlags}
CompCXXFlags   := ${App_CXXFlags}
all: $(BinDir)/$(TARGET)
endif

App_CXXFlags   := $(SharedCompFlags) $(CXXFlags)
IncludeDirs    += $(SrcDir) $(SGX_COMMONDIR) $(PBCRootDir)/include/ $(GMPDir)/
App_Include    := $(addprefix -I, $(IncludeDirs))
App_Link_Flags := $(addprefix -L, $(App_Lib_Dirs)) \
                  $(addprefix -l, $(App_Libs))
App_Cpp_Objs   += $(addprefix $(ObjDir)/, $(Untrusted_Objs)) $(SgxCommonObjs)

ifeq ($(SGX),1)
CompInclude := ${Enclave_Include}
else
CompInclude := ${App_Include}
endif

IBBEC    := $(addprefix $(ObjDir)/, $(IBBEC))
IBBECXX  := $(addprefix $(ObjDir)/, $(IBBECXX))
IBBEObjs := $(addprefix $(ObjDir)/, $(IBBEObjs))
CommonEnclObjs := $(addprefix $(ObjDir)/, $(CommonEnclObjs))
GenericTrustedCpp := $(filter-out $(ObjDir)/$(EnclaveName)_t.o $(IBBEObjs) $(CommonEnclObjs), $(Trusted_Objects))

$(ObjDir)/$(TARGET).o : $(GMPDir)/gmp.h

############## COMMON ##########################################################
$(IBBEC) : $(ObjDir)/%_$(ObjSuffix).o : $(IBBEDir)/%.c
	@${CC} -c $< -o $@ ${CompCFlags} ${CompInclude}
	@echo -e "CC ($(CompString))\t<=\t$<"

$(IBBECXX) : $(ObjDir)/%_$(ObjSuffix).o : $(IBBEDir)/%.cpp
	@${CXX} -c $< -o $@ ${CompCXXFlags} ${CompInclude}
	@echo -e "CXX ($(CompString))\t<=\t$<"
############## UNTRUSTED #######################################################
$(BinDir)/$(TARGET): ${App_Cpp_Objs} ${ExtLibs} | $(BinDir)
	@${CXX} $^ -o $@ ${App_Link_Flags} ${LDFLAGS}
	@echo -e "LINK (App)\t=>\t$@"

$(ObjDir)/%.o : $(SrcDir)/%.c | $(ObjDir)
	@${CC} ${App_CFlags} ${App_Include} -c $< -o $@
	@echo -e "CC (App)\t<=\t$<"

$(ObjDir)/%.o : $(SrcDir)/%.cpp | $(ObjDir)
	@${CXX} ${App_CXXFlags} ${App_Include} -c $< -o $@
	@echo -e "CXX (App)\t<=\t$<"

$(SrcDir)/$(EnclaveName)_u.c : $(SrcDir)/$(EnclaveName).edl $(SGX_EDGER8R)
	@cd $(dir $@) && $(SGX_EDGER8R) --untrusted ../$< --search-path ../$(dir $<) --search-path $(SGX_SDK)/include
	@echo -e "EDGER (App)\t=>\t$@"
	@echo $(GenericTrustedCpp) " <<<" 

$(SgxCommonObjs) : $(ObjDir)/%_u.o : $(SGX_COMMONDIR)/%.cpp | $(ObjDir)
	@${CXX} -c $< -o $@ ${App_CXXFlags} ${App_Include}
	@echo -e "CXX (App)\t<=\t$<"
############## TRUSTED #########################################################
$(BinDir)/$(TARGET).signed.so : %.signed.so : %.so
	@$(SGX_ENCLAVE_SIGNER) sign -enclave $< -config $(SrcDir)/$(EnclaveName).config.xml -out $@ -key $(SrcDir)/enclave-key.pem
	@echo -e "SIGN (Enclave)\t=>\t$@"

$(BinDir)/$(TARGET).so : $(Trusted_Objects)
	@$(CXX) $^ -o $@ $(Trusted_Link_Flags)
	@echo -e "LINK (Enclave)\t=>\t$@"

$(SrcDir)/$(EnclaveName)_t.c : $(SrcDir)/$(EnclaveName).edl $(SGX_EDGER8R)
	@cd $(dir $@) && $(SGX_EDGER8R) --trusted ../$< --search-path ../$(dir $<) --search-path $(SGX_SDK)/include
	@echo -e "EDGER (Enclave)\t=>\t$@"

$(ObjDir)/$(EnclaveName)_t.o : $(SrcDir)/$(EnclaveName)_t.c
	@${CC} -c $< -o $@ ${Enclave_C_Flags} ${Enclave_Include} 
	@echo -e "CC (Enclave)\t<=\t$<"

$(ObjDir)/%_t.o : $(SGX_COMMONDIR)/libc_mock/%.c
	@${CC} -c $< -o $@ ${Enclave_C_Flags} ${Enclave_Include} 
	@echo -e "CC (Enclave)\t<=\t$<"

$(ObjDir)/%_t.o : $(SGX_COMMONDIR)/libc_mock/%.cpp
	@${CXX} -c $< -o $@ ${Enclave_Include} ${Enclave_CXXFlags}
	@echo -e "CXX (Enclave)\t<=\t$<"

$(ObjDir)/%_t.o : $(SGX_COMMONDIR)/%.cpp
	@${CXX} -c $< -o $@ ${Enclave_Include} ${Enclave_CXXFlags}
	@echo -e "CXX (Enclave)\t<=\t$<"

$(GenericTrustedCpp) : $(ObjDir)/%_t.o : $(SrcDir)/%.cpp
	@${CXX} -c $< -o $@ ${Enclave_Include} ${Enclave_CXXFlags}
	@echo -e "CXX (Enclave)\t<=\t$<"
################################################################################
$(PBCLib) : $(GMPLib)
	rm -rf $(PBCDir) && mkdir -p $(PBCDir) && cd $(PBCDir) && LDFLAGS="-L$(GMPDir)/.libs" CPPFLAGS="-I$(PBCRootDir)/include -I$(GMPDir)" ../configure $(LibsConfigFlags) && make

$(GMPLib) : $(GMPDir)/gmp.h
	cd $(GMPDir) && make

$(GMPRootDir)/build-$(SgxToken)/gmp.h :
	rm -rf $(GMPDir) && mkdir -p $(GMPDir) && cd $(GMPDir) && ../configure $(LibsConfigFlags) && sed -i '39,42{s/^\/\///}' gmp.h

$(GMPRootDir)/build-$(NativeToken)/gmp.h :
	rm -rf $(GMPDir) && mkdir -p $(GMPDir) && cd $(GMPDir) && ../configure $(LibsConfigFlags)

################################################################################

$(BinDir):
	@mkdir $(BinDir)

$(ObjDir):
	@mkdir $(ObjDir)

.PHONY: clean clean-libs clean-all list

clean:
	rm -rf $(BinDir) $(ObjDir) $(SrcDir)/*_{t,u}.{c,h} 

clean-libs:
	rm -rf $(PBCDir) $(GMPDir)

clean-all: clean clean-libs

list:
	@$(MAKE) -pRrq -f $(lastword $(MAKEFILE_LIST)) : 2>/dev/null | awk -v RS= -F: '/^# File/,/^# Finished Make data base/ {if ($$1 !~ "^[#.]") {print $$1}}' | sort | egrep -v -e '^[^[:alnum:]]' -e '^$@$$' | xargs

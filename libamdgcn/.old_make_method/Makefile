all::
#
# This is the Makefile for libamdgcn
# using the lightning compiler.
#
# In addition to this Makefile, you need "create_makefile_include.sh"
# and the "SOURCES" file.  SOURCES lists all the source files to compiler
# or assemble. 
# 
# TO see all the generated commands, do not set a value for Verb on next line
#Verb = @

#  This is the directory where the lightning compiler is installed
LCLOC = /opt/amd/llvm

#  This is the libclc git repository directory 
LIBCLC = ./libclc

#  Where is libamdgcn
LIBAMDGCN = .

#  This is where libamdgcn.$MCPU.bc files will be created with "make install"
INSTALLDIR = /opt/rocm/libamdgcn

#  These subdirectories are the only ones modified by "make"
OUTDIR = ./_OUT
BCDIR = $(OUTDIR)/_BC
UTILDIR = $(OUTDIR)/_UTILS
DEPSDIR = $(OUTDIR)/_DEPS

LLVM_CC = $(LCLOC)/bin/clang 
CLFLAGS = -c -target amdgcn--amdhsa -x cl -Xclang -cl-std=CL2.0 -Ucl_khr_fp64 -Ucl_khr_byte_addressable_store -Dcl_clang_storage_class_specifiers  -Dcles_khr_int64 -D__CLC_INTERNAL -emit-llvm -I$(LIBAMDGCN)/libclc_overrides/include -I$(LIBCLC)/generic/include -fno-builtin 

LLVM_AS = $(LCLOC)/bin/llvm-as 
LLVM_LINK = $(LCLOC)/bin/llvm-link
LLVM_OPT = $(LCLOC)/bin/opt

$(UTILDIR)/makefile.kaveri.inc: ./SOURCES
	$(Verb) mkdir -p $(UTILDIR)
	./create_makefile_include.sh $(UTILDIR)/makefile.kaveri.inc kaveri
-include $(UTILDIR)/makefile.kaveri.inc

$(UTILDIR)/makefile.carrizo.inc: ./SOURCES
	$(Verb) mkdir -p $(UTILDIR)
	./create_makefile_include.sh $(UTILDIR)/makefile.carrizo.inc carrizo
-include $(UTILDIR)/makefile.carrizo.inc

$(UTILDIR)/makefile.fiji.inc: ./SOURCES
	$(Verb) mkdir -p $(UTILDIR)
	./create_makefile_include.sh $(UTILDIR)/makefile.fiji.inc fiji
-include $(UTILDIR)/makefile.fiji.inc

includes: $(UTILDIR)/makefile.fiji.inc $(UTILDIR)/makefile.carrizo.inc $(UTILDIR)/makefile.kaveri.inc

$(UTILDIR)/prepare-builtins.o: $(LIBCLC)/utils/prepare-builtins.cpp 
	@echo "Compiling the prepare-builtins.cpp"
	$(Verb) $(LCLOC)/bin/clang++ -pthread -MMD -MF $(DEPSDIR)/prepare-builtins.o.d -I$(LIBAMDGCN)/libclc_overrides/include -I$(LCLOC)/include  -fPIC -fvisibility-inlines-hidden -Wall -W -Wno-unused-parameter -Wwrite-strings -Wcast-qual -Wno-missing-field-initializers -pedantic -Wno-long-long -Wno-uninitialized -Wdelete-non-virtual-dtor -Wno-comment -std=c++11 -ffunction-sections -fdata-sections   -fno-exceptions -fno-rtti -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -fno-exceptions -fno-rtti -c -o $(UTILDIR)/prepare-builtins.o $(LIBCLC)/utils/prepare-builtins.cpp
#  FIX THIS -include utils/prepare-builtins.o.d

$(UTILDIR)/prepare-builtins: $(UTILDIR)/prepare-builtins.o 
	@echo "Linking the prepare-builtins.cpp"
	$(Verb) $(LCLOC)/bin/clang++ -o $(UTILDIR)/prepare-builtins $(UTILDIR)/prepare-builtins.o $(LCLOC)/lib/libLLVMBitWriter.a $(LCLOC)/lib/libLLVMBitReader.a $(LCLOC)/lib/libLLVMCore.a $(LCLOC)/lib/libLLVMSupport.a -pthread -L$(LCLOC)/lib -Wl,-rpath $(LCLOC)/lib -lncurses -ldl

$(UTILDIR)/convert.cl: $(LIBCLC)/generic/lib/gen_convert.py 
	@echo GENERATE convert.cl
	$(Verb) mkdir -p $(UTILDIR)
	python < $(LIBCLC)/generic/lib/gen_convert.py > $(UTILDIR)/convert.cl

#  These targets are generated by create_makefile_include.sh
all:: kaveri carrizo fiji includes

install:
	if [ ! -d $(INSTALLDIR)/lib ] ; then mkdir -p $(INSTALLDIR)/lib ; fi
	cp -p $(BCDIR)/libamdgcn.kaveri.bc $(INSTALLDIR)/lib/.
	cp -p $(BCDIR)/libamdgcn.carrizo.bc $(INSTALLDIR)/lib/.
	cp -p $(BCDIR)/libamdgcn.fiji.bc $(INSTALLDIR)/lib/.
	cp -rp $(LIBCLC)/generic/include  $(INSTALLDIR)/.
	cp -rp $(LIBAMDGCN)/libclc_overrids/include  $(INSTALLDIR)/.

clean:
	if [ -d $(BCDIR) ] ; then rm -r $(BCDIR) ; fi
	if [ -d $(DEPSDIR) ] ; then rm -r $(DEPSDIR) ; fi

distclean: clean
	if [ -d $(UTILDIR) ] ; then rm -r $(UTILDIR) ; fi
	if [ -d $(OUTDIR) ] ; then rm -r $(OUTDIR) ; fi

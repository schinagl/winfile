DEBUG=0

RSC=rc.exe
LINK32=link.exe

# Detect the compiler architecture to create output directories
ARCH=x86
!IF [$(CC) 2>&1 | find "x64" >NUL]==0
ARCH=x64
!ELSE
!IF [$(CC) 2>&1 | find "AMD64" >NUL]==0
ARCH=x64
!ELSE
!IF [$(CC) 2>&1 | find "ARM64" >NUL]==0
ARCH=arm64
!ELSE
!IF [$(CC) 2>&1 | find "Itanium" >NUL]==0
ARCH=ia64
!ELSE
!IF [$(CC) 2>&1 | find "ARM" >NUL]==0
ARCH=arm
!ELSE
!IF [$(CC) 2>&1 | find "MIPS" >NUL]==0
ARCH=mips
!ENDIF # MIPS
!ENDIF # ARM (32)
!ENDIF # Itanium
!ENDIF # ARM64
!ENDIF # AMD64
!ENDIF # x64

# Generate output directory as debug/release followed by arch
!IF $(DEBUG)==1
OUTDIR=.\Debug_$(ARCH)
INTDIR=.\Debug_$(ARCH)
!ELSE
OUTDIR=.\Release_$(ARCH)
INTDIR=.\Release_$(ARCH)
!ENDIF

CPP_OBJS=$(INTDIR)/
BASENAME=VerifyResources

ALL : "$(OUTDIR)\$(BASENAME).exe"

CLEAN : 
    -@rd /s/q $(INTDIR) 2>NUL
    -@rd /s/q $(OUTDIR) 2>NUL

"$(OUTDIR)" :
    @if not exist "$(OUTDIR)/" mkdir "$(OUTDIR)"

CFLAGS=-nologo -MT -W3 -I. 

CDEFS=/D "WIN32" /D "NDEBUG" /D "_WINDOWS" /D "UNICODE" /D "_UNICODE" \
      /D _WIN32_WINNT=0x0600 /D WINVER=0x0600 
LIBS=kernel32.lib user32.lib gdi32.lib comctl32.lib comdlg32.lib advapi32.lib \
     shlwapi.lib shell32.lib uuid.lib version.lib ole32.lib oleaut32.lib
LDFLAGS=-nologo -subsystem:console -incremental:no \
        -out:"$(OUTDIR)/$(BASENAME).exe" -DEBUG -pdb:"$(OUTDIR)/$(BASENAME).pdb"

!IF $(DEBUG)==1
CFLAGS=$(CFLAGS) -Gy -Z7 -Od
!ELSE
CFLAGS=$(CFLAGS) -Gy -Z7 -O1sb1
!ENDIF

#
# Probe for more esoteric aspects of compiler behavior.
# Apparently pragma doesn't cut it for -GS-
#
!IF [$(CC) -GS- 2>&1 | find "unknown" >NUL]>0
CFLAGS=$(CFLAGS) -GS- -GF -EHsc
!ELSE
!IF [$(CC) -GF 2>&1 | find "unknown" >NUL]>0
CFLAGS=$(CFLAGS) -GF -GX
!ELSE
CFLAGS=$(CFLAGS) -Gf -GX
!ENDIF
!ENDIF

#
# Probe for stdcall support.  This should exist on all x86
# compilers, but not x64 compilers.
#

!IF [$(CC) -? 2>&1 | find "/Gz" >NUL]==0
CFLAGS=$(CFLAGS) -Gz
!ENDIF

CFLAGS=$(CFLAGS) $(CDEFS) /Fo"$(INTDIR)/" /c 

OBJS= \
    "$(INTDIR)\VerifyResources.obj"

"$(OUTDIR)\$(BASENAME).exe" : "$(OUTDIR)" $(DEF_FILE) $(OBJS)
    @$(LINK32) @<<
  $(LDFLAGS) $(OBJS) $(LIBS)
<<

.cpp{$(CPP_OBJS)}.obj:
   @$(CC) $(CFLAGS) $(CDEFS) $<  


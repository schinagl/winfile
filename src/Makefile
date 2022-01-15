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
BASENAME=WinFile

ALL : "$(OUTDIR)\$(BASENAME).exe"

CLEAN : 
    -@rd /s/q $(INTDIR) 2>NUL
    -@rd /s/q $(OUTDIR) 2>NUL

"$(OUTDIR)" :
    @if not exist "$(OUTDIR)/" mkdir "$(OUTDIR)"

CFLAGS=-nologo -MT -W3 -I. 

CDEFS=/D "WIN32" /D "NDEBUG" /D "_WINDOWS" /D "FASTMOVE" /D "LFNCLIPBOARD" \
      /D "MEMDOUBLE" /D "STRICT" /D "UNICODE" /D "_UNICODE" /D "USELASTDOT" \
      /D _WIN32_WINNT=0x0500 /D WINVER=0x0500 
LIBS=kernel32.lib user32.lib gdi32.lib comctl32.lib comdlg32.lib advapi32.lib \
     shlwapi.lib shell32.lib uuid.lib version.lib ole32.lib oleaut32.lib
LDFLAGS=-nologo -subsystem:windows -incremental:no \
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

RSC_PROJ=/fo"$(INTDIR)/res.res" /d "NDEBUG" 

OBJS= \
    "$(INTDIR)\dbg.obj" \
    "$(INTDIR)\lfn.obj" \
    "$(INTDIR)\lfnmisc.obj" \
    "$(INTDIR)\numfmt.obj" \
    "$(INTDIR)\res.res" \
    "$(INTDIR)\suggest.obj" \
    "$(INTDIR)\tbar.obj" \
    "$(INTDIR)\treectl.obj" \
    "$(INTDIR)\wfassoc.obj" \
    "$(INTDIR)\wfchgnot.obj" \
    "$(INTDIR)\wfcomman.obj" \
    "$(INTDIR)\wfcopy.obj" \
    "$(INTDIR)\wfdir.obj" \
    "$(INTDIR)\wfdirrd.obj" \
    "$(INTDIR)\wfdirsrc.obj" \
    "$(INTDIR)\wfdlgs.obj" \
    "$(INTDIR)\wfdlgs2.obj" \
    "$(INTDIR)\wfdlgs3.obj" \
    "$(INTDIR)\wfdos.obj" \
    "$(INTDIR)\wfdrives.obj" \
    "$(INTDIR)\wfdrop.obj" \
    "$(INTDIR)\wfext.obj" \
    "$(INTDIR)\wffile.obj" \
    "$(INTDIR)\wfinfo.obj" \
    "$(INTDIR)\wfinit.obj" \
    "$(INTDIR)\wfloc.obj" \
    "$(INTDIR)\wfmem.obj" \
    "$(INTDIR)\wfprint.obj" \
    "$(INTDIR)\wfsearch.obj" \
    "$(INTDIR)\wftree.obj" \
    "$(INTDIR)\wfutil.obj" \
    "$(INTDIR)\winfile.obj" \
    "$(INTDIR)\wnetcaps.obj"

RCFILES=                     \
    "rescmn.rc"              \
    "lang\res_de-DE.rc"      \
    "lang\res_en-US.rc"      \
    "lang\res_he-IL.rc"      \
    "lang\res_ja-JP.rc"      \
    "lang\res_pl-PL.rc"      \
    "lang\res_tr-TR.rc"      \
    "lang\res_zh-CN.rc"      \
    "lang\suggest_de-DE.db"  \
    "lang\suggest_en-US.db"  \
    "lang\suggest_he-IL.db"  \
    "lang\suggest_ja-JP.db"  \
    "lang\suggest_pl-PL.db"  \
    "lang\suggest_tr-TR.db"  \
    "lang\suggest_zh-CN.db"  \
    "lang\winfile_de-DE.dlg" \
    "lang\winfile_en-US.dlg" \
    "lang\winfile_he-IL.dlg" \
    "lang\winfile_ja-JP.dlg" \
    "lang\winfile_pl-PL.dlg" \
    "lang\winfile_tr-TR.dlg" \
    "lang\winfile_zh-CN.dlg"

"$(OUTDIR)\$(BASENAME).exe" : "$(OUTDIR)" $(DEF_FILE) $(OBJS)
    @$(LINK32) @<<
  $(LDFLAGS) $(OBJS) $(LIBS)
<<

.c{$(CPP_OBJS)}.obj:
   @$(CC) $(CFLAGS) $(CDEFS) $<  

# Check if the resource compiler has MUI support, and build appropriate
# resources.  This dance is done because RC doesn't appear to handle
# pragma within #ifdef.
!IF [$(RC) -g 2>&1 | find "missing default" >NUL]==0
"$(INTDIR)\res.res" : resmui.rc $(RCFILES)
   @$(RSC) $(RSC_PROJ) resmui.rc
!ELSE
"$(INTDIR)\res.res" : resnomui.rc $(RCFILES)
   @$(RSC) $(RSC_PROJ) resnomui.rc
!ENDIF


cmake_minimum_required(VERSION 3.13)
project(uqm VERSION 0.8.0 LANGUAGES C)

# ============================================================================
# Android NDK 交叉编译 sc2-uqm -> libuqm.so
# 参考 dholmes215/uqm (cmake branch) 的 CMake 构建系统
# ============================================================================

set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/sc2/src)

# --- 构建选项 ---
option(UQM_NETPLAY "启用网络对战" OFF)
option(UQM_USE_INTERNAL_MIKMOD "使用内置 mikmod" ON)
option(UQM_USE_ZIP_IO "启用 zip 资源读取" ON)
option(UQM_USE_OGGVORBIS "启用 Ogg Vorbis 音频" ON)

# --- 查找依赖 ---
find_package(SDL2 REQUIRED)
find_package(ZLIB REQUIRED)
find_package(PNG REQUIRED)

if(UQM_USE_OGGVORBIS)
    find_package(Ogg QUIET)
    find_package(Vorbis QUIET)
endif()

# --- OpenGL (gl4es) ---
# 在 Docker 构建中通过 -DOPENGL_opengl_LIBRARY 和 -DOPENGL_glx_LIBRARY 传入
find_package(OpenGL QUIET)

# --- 生成 config_unix.h ---
# Android 是 Unix-like 系统，config.h 的 #else 分支会 #include "config_unix.h"
# 我们直接生成一个适合 Android 的 config_unix.h
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config_unix_android.h.in
    ${SRC_DIR}/config_unix.h
    @ONLY
)

# ============================================================================
# 源文件收集
# ============================================================================

# --- 顶层源文件 ---
set(TOP_SOURCES
    ${SRC_DIR}/options.c
    ${SRC_DIR}/port.c
    ${SRC_DIR}/uqm.c
)

# --- uqm 游戏核心 ---
set(UQM_SOURCES
    ${SRC_DIR}/uqm/battle.c
    ${SRC_DIR}/uqm/battlecontrols.c
    ${SRC_DIR}/uqm/border.c
    ${SRC_DIR}/uqm/build.c
    ${SRC_DIR}/uqm/cleanup.c
    ${SRC_DIR}/uqm/clock.c
    ${SRC_DIR}/uqm/cnctdlg.c
    ${SRC_DIR}/uqm/collide.c
    ${SRC_DIR}/uqm/comm.c
    ${SRC_DIR}/uqm/commanim.c
    ${SRC_DIR}/uqm/commglue.c
    ${SRC_DIR}/uqm/confirm.c
    ${SRC_DIR}/uqm/credits.c
    ${SRC_DIR}/uqm/cyborg.c
    ${SRC_DIR}/uqm/demo.c
    ${SRC_DIR}/uqm/displist.c
    ${SRC_DIR}/uqm/dummy.c
    ${SRC_DIR}/uqm/encount.c
    ${SRC_DIR}/uqm/flash.c
    ${SRC_DIR}/uqm/fmv.c
    ${SRC_DIR}/uqm/galaxy.c
    ${SRC_DIR}/uqm/gameev.c
    ${SRC_DIR}/uqm/gameinp.c
    ${SRC_DIR}/uqm/gameopt.c
    ${SRC_DIR}/uqm/gendef.c
    ${SRC_DIR}/uqm/getchar.c
    ${SRC_DIR}/uqm/globdata.c
    ${SRC_DIR}/uqm/gravity.c
    ${SRC_DIR}/uqm/cons_res.c
    ${SRC_DIR}/uqm/grpinfo.c
    ${SRC_DIR}/uqm/hyper.c
    ${SRC_DIR}/uqm/init.c
    ${SRC_DIR}/uqm/intel.c
    ${SRC_DIR}/uqm/intro.c
    ${SRC_DIR}/uqm/ipdisp.c
    ${SRC_DIR}/uqm/load.c
    ${SRC_DIR}/uqm/loadmele.c
    ${SRC_DIR}/uqm/loadship.c
    ${SRC_DIR}/uqm/master.c
    ${SRC_DIR}/uqm/melee.c
    ${SRC_DIR}/uqm/menu.c
    ${SRC_DIR}/uqm/misc.c
    ${SRC_DIR}/uqm/oscill.c
    ${SRC_DIR}/uqm/outfit.c
    ${SRC_DIR}/uqm/pickmele.c
    ${SRC_DIR}/uqm/pickship.c
    ${SRC_DIR}/uqm/plandata.c
    ${SRC_DIR}/uqm/process.c
    ${SRC_DIR}/uqm/restart.c
    ${SRC_DIR}/uqm/save.c
    ${SRC_DIR}/uqm/settings.c
    ${SRC_DIR}/uqm/setup.c
    ${SRC_DIR}/uqm/setupmenu.c
    ${SRC_DIR}/uqm/ship.c
    ${SRC_DIR}/uqm/shipstat.c
    ${SRC_DIR}/uqm/shipyard.c
    ${SRC_DIR}/uqm/sis.c
    ${SRC_DIR}/uqm/sounds.c
    ${SRC_DIR}/uqm/starbase.c
    ${SRC_DIR}/uqm/starcon.c
    ${SRC_DIR}/uqm/starmap.c
    ${SRC_DIR}/uqm/state.c
    ${SRC_DIR}/uqm/status.c
    ${SRC_DIR}/uqm/tactrans.c
    ${SRC_DIR}/uqm/trans.c
    ${SRC_DIR}/uqm/uqmdebug.c
    ${SRC_DIR}/uqm/util.c
    ${SRC_DIR}/uqm/velocity.c
    ${SRC_DIR}/uqm/weapon.c
)

# --- uqm/comm 各外星种族对话 ---
set(UQM_COMM_SOURCES
    ${SRC_DIR}/uqm/comm/arilou/arilouc.c
    ${SRC_DIR}/uqm/comm/blackur/blackurc.c
    ${SRC_DIR}/uqm/comm/chmmr/chmmrc.c
    ${SRC_DIR}/uqm/comm/comandr/comandr.c
    ${SRC_DIR}/uqm/comm/druuge/druugec.c
    ${SRC_DIR}/uqm/comm/ilwrath/ilwrathc.c
    ${SRC_DIR}/uqm/comm/melnorm/melnorm.c
    ${SRC_DIR}/uqm/comm/mycon/myconc.c
    ${SRC_DIR}/uqm/comm/orz/orzc.c
    ${SRC_DIR}/uqm/comm/pkunk/pkunkc.c
    ${SRC_DIR}/uqm/comm/rebel/rebel.c
    ${SRC_DIR}/uqm/comm/shofixt/shofixt.c
    ${SRC_DIR}/uqm/comm/slyhome/slyhome.c
    ${SRC_DIR}/uqm/comm/slyland/slyland.c
    ${SRC_DIR}/uqm/comm/spahome/spahome.c
    ${SRC_DIR}/uqm/comm/spathi/spathic.c
    ${SRC_DIR}/uqm/comm/starbas/starbas.c
    ${SRC_DIR}/uqm/comm/supox/supoxc.c
    ${SRC_DIR}/uqm/comm/syreen/syreenc.c
    ${SRC_DIR}/uqm/comm/talkpet/talkpet.c
    ${SRC_DIR}/uqm/comm/thradd/thraddc.c
    ${SRC_DIR}/uqm/comm/umgah/umgahc.c
    ${SRC_DIR}/uqm/comm/urquan/urquanc.c
    ${SRC_DIR}/uqm/comm/utwig/utwigc.c
    ${SRC_DIR}/uqm/comm/vux/vuxc.c
    ${SRC_DIR}/uqm/comm/yehat/yehatc.c
    ${SRC_DIR}/uqm/comm/zoqfot/zoqfotc.c
)

# --- uqm/planets ---
set(UQM_PLANETS_SOURCES
    ${SRC_DIR}/uqm/planets/calc.c
    ${SRC_DIR}/uqm/planets/cargo.c
    ${SRC_DIR}/uqm/planets/devices.c
    ${SRC_DIR}/uqm/planets/genburv.c
    ${SRC_DIR}/uqm/planets/genchmmr.c
    ${SRC_DIR}/uqm/planets/gencol.c
    ${SRC_DIR}/uqm/planets/gendru.c
    ${SRC_DIR}/uqm/planets/genilw.c
    ${SRC_DIR}/uqm/planets/genmel.c
    ${SRC_DIR}/uqm/planets/genmothark.c
    ${SRC_DIR}/uqm/planets/genmyc.c
    ${SRC_DIR}/uqm/planets/gennegl.c
    ${SRC_DIR}/uqm/planets/genorz.c
    ${SRC_DIR}/uqm/planets/genpet.c
    ${SRC_DIR}/uqm/planets/genpku.c
    ${SRC_DIR}/uqm/planets/genprecrsr.c
    ${SRC_DIR}/uqm/planets/genrain.c
    ${SRC_DIR}/uqm/planets/gensam.c
    ${SRC_DIR}/uqm/planets/genshof.c
    ${SRC_DIR}/uqm/planets/gensly.c
    ${SRC_DIR}/uqm/planets/gensol.c
    ${SRC_DIR}/uqm/planets/genspa.c
    ${SRC_DIR}/uqm/planets/gensup.c
    ${SRC_DIR}/uqm/planets/gensyr.c
    ${SRC_DIR}/uqm/planets/genthrad.c
    ${SRC_DIR}/uqm/planets/gentopo.c
    ${SRC_DIR}/uqm/planets/genurq.c
    ${SRC_DIR}/uqm/planets/genutw.c
    ${SRC_DIR}/uqm/planets/genvault.c
    ${SRC_DIR}/uqm/planets/genvux.c
    ${SRC_DIR}/uqm/planets/genwreck.c
    ${SRC_DIR}/uqm/planets/genyeh.c
    ${SRC_DIR}/uqm/planets/genzoq.c
    ${SRC_DIR}/uqm/planets/lander.c
    ${SRC_DIR}/uqm/planets/orbits.c
    ${SRC_DIR}/uqm/planets/oval.c
    ${SRC_DIR}/uqm/planets/pl_stuff.c
    ${SRC_DIR}/uqm/planets/planets.c
    ${SRC_DIR}/uqm/planets/plangen.c
    ${SRC_DIR}/uqm/planets/pstarmap.c
    ${SRC_DIR}/uqm/planets/report.c
    ${SRC_DIR}/uqm/planets/roster.c
    ${SRC_DIR}/uqm/planets/scan.c
    ${SRC_DIR}/uqm/planets/solarsys.c
    ${SRC_DIR}/uqm/planets/surface.c
)

# --- uqm/ships ---
set(UQM_SHIPS_SOURCES
    ${SRC_DIR}/uqm/ships/androsyn/androsyn.c
    ${SRC_DIR}/uqm/ships/arilou/arilou.c
    ${SRC_DIR}/uqm/ships/blackurq/blackurq.c
    ${SRC_DIR}/uqm/ships/chenjesu/chenjesu.c
    ${SRC_DIR}/uqm/ships/chmmr/chmmr.c
    ${SRC_DIR}/uqm/ships/druuge/druuge.c
    ${SRC_DIR}/uqm/ships/human/human.c
    ${SRC_DIR}/uqm/ships/ilwrath/ilwrath.c
    ${SRC_DIR}/uqm/ships/lastbat/lastbat.c
    ${SRC_DIR}/uqm/ships/melnorme/melnorme.c
    ${SRC_DIR}/uqm/ships/mmrnmhrm/mmrnmhrm.c
    ${SRC_DIR}/uqm/ships/mycon/mycon.c
    ${SRC_DIR}/uqm/ships/orz/orz.c
    ${SRC_DIR}/uqm/ships/pkunk/pkunk.c
    ${SRC_DIR}/uqm/ships/probe/probe.c
    ${SRC_DIR}/uqm/ships/shofixti/shofixti.c
    ${SRC_DIR}/uqm/ships/sis_ship/sis_ship.c
    ${SRC_DIR}/uqm/ships/slylandr/slylandr.c
    ${SRC_DIR}/uqm/ships/spathi/spathi.c
    ${SRC_DIR}/uqm/ships/supox/supox.c
    ${SRC_DIR}/uqm/ships/syreen/syreen.c
    ${SRC_DIR}/uqm/ships/thradd/thradd.c
    ${SRC_DIR}/uqm/ships/umgah/umgah.c
    ${SRC_DIR}/uqm/ships/urquan/urquan.c
    ${SRC_DIR}/uqm/ships/utwig/utwig.c
    ${SRC_DIR}/uqm/ships/vux/vux.c
    ${SRC_DIR}/uqm/ships/yehat/yehat.c
    ${SRC_DIR}/uqm/ships/zoqfot/zoqfot.c
)

# --- libs/ 引擎库 ---
set(LIBS_SOURCES
    # callback
    ${SRC_DIR}/libs/callback/alarm.c
    ${SRC_DIR}/libs/callback/callback.c
    # decomp
    ${SRC_DIR}/libs/decomp/lzdecode.c
    ${SRC_DIR}/libs/decomp/lzencode.c
    ${SRC_DIR}/libs/decomp/update.c
    # file
    ${SRC_DIR}/libs/file/dirs.c
    ${SRC_DIR}/libs/file/files.c
    ${SRC_DIR}/libs/file/temp.c
    # graphics (core)
    ${SRC_DIR}/libs/graphics/boxint.c
    ${SRC_DIR}/libs/graphics/clipline.c
    ${SRC_DIR}/libs/graphics/cmap.c
    ${SRC_DIR}/libs/graphics/context.c
    ${SRC_DIR}/libs/graphics/drawable.c
    ${SRC_DIR}/libs/graphics/filegfx.c
    ${SRC_DIR}/libs/graphics/font.c
    ${SRC_DIR}/libs/graphics/frame.c
    ${SRC_DIR}/libs/graphics/gfx_common.c
    ${SRC_DIR}/libs/graphics/intersec.c
    ${SRC_DIR}/libs/graphics/loaddisp.c
    ${SRC_DIR}/libs/graphics/pixmap.c
    ${SRC_DIR}/libs/graphics/resgfx.c
    ${SRC_DIR}/libs/graphics/tfb_draw.c
    ${SRC_DIR}/libs/graphics/tfb_prim.c
    ${SRC_DIR}/libs/graphics/widgets.c
    # graphics/sdl
    ${SRC_DIR}/libs/graphics/sdl/2xscalers.c
    ${SRC_DIR}/libs/graphics/sdl/3do_blt.c
    ${SRC_DIR}/libs/graphics/sdl/3do_funcs.c
    ${SRC_DIR}/libs/graphics/sdl/3do_getbody.c
    ${SRC_DIR}/libs/graphics/sdl/bbox.c
    ${SRC_DIR}/libs/graphics/sdl/biadv2x.c
    ${SRC_DIR}/libs/graphics/sdl/bilinear2x.c
    ${SRC_DIR}/libs/graphics/sdl/canvas.c
    ${SRC_DIR}/libs/graphics/sdl/dcqueue.c
    ${SRC_DIR}/libs/graphics/sdl/hq2x.c
    ${SRC_DIR}/libs/graphics/sdl/nearest2x.c
    ${SRC_DIR}/libs/graphics/sdl/opengl.c
    ${SRC_DIR}/libs/graphics/sdl/primitives.c
    ${SRC_DIR}/libs/graphics/sdl/pure.c
    ${SRC_DIR}/libs/graphics/sdl/rndzoom.c
    ${SRC_DIR}/libs/graphics/sdl/rotozoom.c
    ${SRC_DIR}/libs/graphics/sdl/scalers.c
    ${SRC_DIR}/libs/graphics/sdl/sdl_common.c
    ${SRC_DIR}/libs/graphics/sdl/sdluio.c
    ${SRC_DIR}/libs/graphics/sdl/triscan2x.c
    # heap
    ${SRC_DIR}/libs/heap/heap.c
    # input
    ${SRC_DIR}/libs/input/input_common.c
    ${SRC_DIR}/libs/input/sdl/input.c
    ${SRC_DIR}/libs/input/sdl/keynames.c
    ${SRC_DIR}/libs/input/sdl/vcontrol.c
    # list
    ${SRC_DIR}/libs/list/list.c
    # log (使用 stub，不使用 macOS/Windows 弹窗)
    ${SRC_DIR}/libs/log/uqmlog.c
    ${SRC_DIR}/libs/log/msgbox_stub.c
    # math
    ${SRC_DIR}/libs/math/random.c
    ${SRC_DIR}/libs/math/random2.c
    ${SRC_DIR}/libs/math/sqrt.c
    # md5
    ${SRC_DIR}/libs/md5/md5.c
    # memory
    ${SRC_DIR}/libs/memory/w_memlib.c
    # resource
    ${SRC_DIR}/libs/resource/direct.c
    ${SRC_DIR}/libs/resource/filecntl.c
    ${SRC_DIR}/libs/resource/getres.c
    ${SRC_DIR}/libs/resource/loadres.c
    ${SRC_DIR}/libs/resource/propfile.c
    ${SRC_DIR}/libs/resource/resinit.c
    ${SRC_DIR}/libs/resource/stringbank.c
    # sound (core)
    ${SRC_DIR}/libs/sound/audiocore.c
    ${SRC_DIR}/libs/sound/fileinst.c
    ${SRC_DIR}/libs/sound/music.c
    ${SRC_DIR}/libs/sound/resinst.c
    ${SRC_DIR}/libs/sound/sfx.c
    ${SRC_DIR}/libs/sound/sound.c
    ${SRC_DIR}/libs/sound/stream.c
    ${SRC_DIR}/libs/sound/trackplayer.c
    # sound/decoders
    ${SRC_DIR}/libs/sound/decoders/aiffaud.c
    ${SRC_DIR}/libs/sound/decoders/decoder.c
    ${SRC_DIR}/libs/sound/decoders/dukaud.c
    ${SRC_DIR}/libs/sound/decoders/modaud.c
    ${SRC_DIR}/libs/sound/decoders/wav.c
    # sound/mixer (SDL)
    ${SRC_DIR}/libs/sound/mixer/mixer.c
    ${SRC_DIR}/libs/sound/mixer/sdl/audiodrv_sdl.c
    ${SRC_DIR}/libs/sound/mixer/nosound/audiodrv_nosound.c
    # strings
    ${SRC_DIR}/libs/strings/getstr.c
    ${SRC_DIR}/libs/strings/sfileins.c
    ${SRC_DIR}/libs/strings/sresins.c
    ${SRC_DIR}/libs/strings/strings.c
    ${SRC_DIR}/libs/strings/unicode.c
    # task
    ${SRC_DIR}/libs/task/tasklib.c
    # threads (SDL for Android)
    ${SRC_DIR}/libs/threads/thrcommon.c
    ${SRC_DIR}/libs/threads/sdl/sdlthreads.c
    # time (SDL for Android)
    ${SRC_DIR}/libs/time/timecommon.c
    ${SRC_DIR}/libs/time/sdl/sdltime.c
    # uio
    ${SRC_DIR}/libs/uio/charhashtable.c
    ${SRC_DIR}/libs/uio/debug.c
    ${SRC_DIR}/libs/uio/defaultfs.c
    ${SRC_DIR}/libs/uio/fileblock.c
    ${SRC_DIR}/libs/uio/fstypes.c
    ${SRC_DIR}/libs/uio/gphys.c
    ${SRC_DIR}/libs/uio/hashtable.c
    ${SRC_DIR}/libs/uio/io.c
    ${SRC_DIR}/libs/uio/ioaux.c
    ${SRC_DIR}/libs/uio/match.c
    ${SRC_DIR}/libs/uio/memdebug.c
    ${SRC_DIR}/libs/uio/mount.c
    ${SRC_DIR}/libs/uio/mounttree.c
    ${SRC_DIR}/libs/uio/paths.c
    ${SRC_DIR}/libs/uio/physical.c
    ${SRC_DIR}/libs/uio/stdio/stdio.c
    ${SRC_DIR}/libs/uio/uiostream.c
    ${SRC_DIR}/libs/uio/uioutils.c
    ${SRC_DIR}/libs/uio/utils.c
    # video
    ${SRC_DIR}/libs/video/dukvid.c
    ${SRC_DIR}/libs/video/legacyplayer.c
    ${SRC_DIR}/libs/video/vfileins.c
    ${SRC_DIR}/libs/video/video.c
    ${SRC_DIR}/libs/video/videodec.c
    ${SRC_DIR}/libs/video/vidplayer.c
    ${SRC_DIR}/libs/video/vresins.c
)

# --- 条件源文件 ---

# Ogg Vorbis 解码器
if(UQM_USE_OGGVORBIS)
    list(APPEND LIBS_SOURCES ${SRC_DIR}/libs/sound/decoders/oggaud.c)
endif()

# ZIP I/O 支持
if(UQM_USE_ZIP_IO)
    list(APPEND LIBS_SOURCES ${SRC_DIR}/libs/uio/zip/zip.c)
endif()

# 内置 mikmod
if(UQM_USE_INTERNAL_MIKMOD)
    set(MIKMOD_SOURCES
        ${SRC_DIR}/libs/mikmod/drv_nos.c
        ${SRC_DIR}/libs/mikmod/load_it.c
        ${SRC_DIR}/libs/mikmod/load_mod.c
        ${SRC_DIR}/libs/mikmod/load_s3m.c
        ${SRC_DIR}/libs/mikmod/load_stm.c
        ${SRC_DIR}/libs/mikmod/load_xm.c
        ${SRC_DIR}/libs/mikmod/mdreg.c
        ${SRC_DIR}/libs/mikmod/mdriver.c
        ${SRC_DIR}/libs/mikmod/mloader.c
        ${SRC_DIR}/libs/mikmod/mlreg.c
        ${SRC_DIR}/libs/mikmod/mlutil.c
        ${SRC_DIR}/libs/mikmod/mmalloc.c
        ${SRC_DIR}/libs/mikmod/mmerror.c
        ${SRC_DIR}/libs/mikmod/mmio.c
        ${SRC_DIR}/libs/mikmod/mplayer.c
        ${SRC_DIR}/libs/mikmod/munitrk.c
        ${SRC_DIR}/libs/mikmod/mwav.c
        ${SRC_DIR}/libs/mikmod/npertab.c
        ${SRC_DIR}/libs/mikmod/sloader.c
        ${SRC_DIR}/libs/mikmod/virtch.c
        ${SRC_DIR}/libs/mikmod/virtch2.c
        ${SRC_DIR}/libs/mikmod/virtch_common.c
    )
endif()

# Netplay 源文件（默认关闭）
if(UQM_NETPLAY)
    set(NETPLAY_SOURCES
        ${SRC_DIR}/uqm/netplay/checkbuf.c
        ${SRC_DIR}/uqm/netplay/checksum.c
        ${SRC_DIR}/uqm/netplay/crc.c
        ${SRC_DIR}/uqm/netplay/netconnection.c
        ${SRC_DIR}/uqm/netplay/netinput.c
        ${SRC_DIR}/uqm/netplay/netmelee.c
        ${SRC_DIR}/uqm/netplay/netmisc.c
        ${SRC_DIR}/uqm/netplay/netoptions.c
        ${SRC_DIR}/uqm/netplay/netrcv.c
        ${SRC_DIR}/uqm/netplay/netsend.c
        ${SRC_DIR}/uqm/netplay/netstate.c
        ${SRC_DIR}/uqm/netplay/notify.c
        ${SRC_DIR}/uqm/netplay/packet.c
        ${SRC_DIR}/uqm/netplay/packethandlers.c
        ${SRC_DIR}/uqm/netplay/packetq.c
        ${SRC_DIR}/uqm/netplay/packetsenders.c
        ${SRC_DIR}/uqm/netplay/proto/npconfirm.c
        ${SRC_DIR}/uqm/netplay/proto/ready.c
        ${SRC_DIR}/uqm/netplay/proto/reset.c
        # network libs
        ${SRC_DIR}/libs/network/netport.c
        ${SRC_DIR}/libs/network/network_bsd.c
        ${SRC_DIR}/libs/network/connect/connect.c
        ${SRC_DIR}/libs/network/connect/listen.c
        ${SRC_DIR}/libs/network/connect/resolve.c
        ${SRC_DIR}/libs/network/netmanager/netmanager_bsd.c
        ${SRC_DIR}/libs/network/netmanager/ndesc.c
        ${SRC_DIR}/libs/network/socket/socket.c
        ${SRC_DIR}/libs/network/socket/socket_bsd.c
    )
endif()

# getopt (Android NDK bionic 没有 getopt_long)
set(GETOPT_SOURCES
    ${SRC_DIR}/getopt/getopt.c
    ${SRC_DIR}/getopt/getopt1.c
)

# regex
set(REGEX_SOURCES
    ${SRC_DIR}/regex/regex.c
)

# ============================================================================
# 构建共享库 libuqm.so
# ============================================================================
add_library(uqm SHARED
    ${TOP_SOURCES}
    ${UQM_SOURCES}
    ${UQM_COMM_SOURCES}
    ${UQM_PLANETS_SOURCES}
    ${UQM_SHIPS_SOURCES}
    ${LIBS_SOURCES}
    ${GETOPT_SOURCES}
    ${REGEX_SOURCES}
)

if(UQM_USE_INTERNAL_MIKMOD)
    target_sources(uqm PRIVATE ${MIKMOD_SOURCES})
endif()

if(UQM_NETPLAY)
    target_sources(uqm PRIVATE ${NETPLAY_SOURCES})
endif()

# --- Include 路径 ---
target_include_directories(uqm PRIVATE
    ${SRC_DIR}
    ${SRC_DIR}/libs
    ${SRC_DIR}/libs/mikmod
)

# --- 编译定义 ---
target_compile_definitions(uqm PRIVATE
    # 图形模块
    GFXMODULE_SDL
    # SDL 方言 (SDL2)
    SDL_DIR=SDL2
    # 线程库
    THREADLIB_SDL
    # 音频混合器
    SOUNDMODULE=SOUNDMODULE_SDL
    # I/O 格式
    HAVE_JOYSTICK
    # 不定义 HAVE_GETOPT_LONG — 使用内置 getopt 源文件
    # battlecontrols.c 等文件直接使用 NUM_PLAYERS 但不 include init.h
    NUM_PLAYERS=2
)

# --- 编译选项 ---
# NDK clang 默认开启 -Werror=format-security，但 uqm 的 log_add 传变量作为格式字符串
target_compile_options(uqm PRIVATE -Wno-error=format-security)

if(UQM_USE_ZIP_IO)
    target_compile_definitions(uqm PRIVATE USE_ZIP_IO)
endif()

if(UQM_USE_OGGVORBIS)
    target_compile_definitions(uqm PRIVATE OGGVORBIS)
endif()

if(UQM_USE_INTERNAL_MIKMOD)
    target_compile_definitions(uqm PRIVATE USE_INTERNAL_MIKMOD)
endif()

if(UQM_NETPLAY)
    target_compile_definitions(uqm PRIVATE NETPLAY=NETPLAY_FULL)
endif()

# --- 链接依赖 ---
# SDL2
if(TARGET SDL2::SDL2)
    target_link_libraries(uqm PRIVATE SDL2::SDL2)
else()
    # Android NDK toolchain 可能不提供 SDL2::SDL2 target
    target_include_directories(uqm PRIVATE ${SDL2_INCLUDE_DIRS})
    target_link_libraries(uqm PRIVATE ${SDL2_LIBRARIES})
endif()

# zlib
target_link_libraries(uqm PRIVATE ZLIB::ZLIB)

# libpng
target_link_libraries(uqm PRIVATE PNG::PNG)

# Ogg/Vorbis
if(UQM_USE_OGGVORBIS)
    if(TARGET Vorbis::vorbisfile)
        target_link_libraries(uqm PRIVATE Vorbis::vorbisfile Vorbis::vorbis Ogg::ogg)
    elseif(VORBISFILE_FOUND OR VORBIS_FOUND)
        target_link_libraries(uqm PRIVATE ${VORBIS_LIBRARIES} ${OGG_LIBRARIES})
        target_include_directories(uqm PRIVATE ${VORBIS_INCLUDE_DIRS} ${OGG_INCLUDE_DIRS})
    else()
        # 尝试 pkg-config
        include(FindPkgConfig)
        pkg_check_modules(VORBIS QUIET vorbis vorbisfile ogg)
        if(VORBIS_FOUND)
            target_link_libraries(uqm PRIVATE ${VORBIS_LINK_LIBRARIES})
            target_include_directories(uqm PRIVATE ${VORBIS_INCLUDE_DIRS})
        else()
            message(WARNING "Ogg/Vorbis not found, disabling OGGVORBIS support")
            target_compile_definitions(uqm PRIVATE OVCODEC_NONE)
        endif()
    endif()
endif()

# OpenGL (gl4es on Android)
if(OPENGL_opengl_LIBRARY)
    target_include_directories(uqm PRIVATE ${OPENGL_opengl_LIBRARY})
endif()
if(OPENGL_glx_LIBRARY)
    target_link_libraries(uqm PRIVATE ${OPENGL_glx_LIBRARY})
endif()

# Android 系统库
if(ANDROID)
    find_library(LOG_LIB log)
    find_library(ANDROID_LIB android)
    target_link_libraries(uqm PRIVATE ${LOG_LIB} ${ANDROID_LIB})
endif()

# 数学库
target_link_libraries(uqm PRIVATE m)

# 设置输出名（libuqm.so）
set_target_properties(uqm PROPERTIES
    OUTPUT_NAME "uqm"
    # 不添加版本后缀
    VERSION ""
    SOVERSION ""
)


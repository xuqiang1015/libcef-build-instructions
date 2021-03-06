CEF编译指南

1. 概述
	     CEF版本：2623
   	Chromium版本：49.0.2623.95
	    运行平台：Window XP SP3+

2. Windows编译环境
	* Win7+，64-bit, 内存8G+, 硬盘40GB+
	* Visual Studio VS2014 update4
	* Windows 10 SDK
	* VPN
	* Python
	* Chromium部署工具depot_tools和gclient
	* Cygwin命令行工具（也可以用其他命令行工具）

3. 安装depot_tools
	3.1 下载工具包
		wget https://storage.googleapis.com/chrome-infra/depot_tools.zip
	3.2 解压depot_tools.zip到一个指定目录（例如：D:\depot_tools)
	3.3 添加D:\depot_tools到环境变量PATH

4. 准备Chromium源码
	4.1 下载源码
		wget https://gsdview.appspot.com/chromium-browser-official/chromium-49.0.2623.110.tar.xz
	4.2 解压源码
		mkdir -p chromium/
		tar Jxvf chromium-49.0.2623.110.tar.xz -C chromium/src
		cd chromium
		mv chromium-49.0.2623.110 src

5. 准备CEF源码
	5.1 下载源码
		cd chromium/src
		git clone https://bitbucket.org/chromiumembedded/cef.git
		git checkout -t origin/2623

6. 准备Chromium依赖
	6.1 下载Bison
		cd chromium/src/third_party
		git clone https://chromium.googlesource.com/chromium/deps/bison
	6.2 下载gperf
		cd chromium/src/third_party
		git clone https://chromium.googlesource.com/chromium/deps/gperf
	6.3 下载yasm.exe，放到chromium\src\third_party\yasm\binaries\win目录下
	6.4 下载d3dcompiler_47.dll文件，放到%VS_ROOT%\Redist\d3d\x86目录下
	6.5 添加缺失文件（chromium\src\chrome\test\data\webui\i18n_process_css_test.html）

		<!doctype html>
		<style>
		<include src="../../../../ui/webui/resources/css/i18n_process.css">
		</style>
		<h1 i18n-content="buy"></h1>
		<span i18n-values=".innerHTML:link"></span>
		<script>
		function testI18nProcess_NbspPlaceholder() {
		  var h1 = document.querySelector('h1');
		  var span = document.querySelector('span');
		  assertFalse(document.documentElement.hasAttribute('i18n-processed'));
		  assertEquals('', h1.textContent);
		  assertEquals('', span.textContent);
		  /* We can't check that the non-breaking space hack actually works because it
		   * uses :psuedo-elements that are inaccessible to the DOM. Let's just check
		   * that they're not auto-collapsed. */
		  assertNotEqual(0, h1.offsetHeight);
		  assertNotEqual(0, span.offsetHeight);
		  h1.removeAttribute('i18n-content');
		  assertEquals(0, h1.offsetHeight);
		  span.removeAttribute('i18n-values');
		  assertEquals(0, span.offsetHeight);
		}
		</script>

7. 去掉编译选项'/WX'，禁止把警告当做错误处理
	注释掉chromium\src\tools\gyp\pylib\gyp\msvs_emulation.py关于'/WX'的内容:
	"""cl('WarnAsError', map={'true': '/WX'})"""

8. 准备工程配置脚本
	8.1 创建一个批处理文件，放入'chromium/src/cef'目录中（create_projects.bat)
	8.2 将以下内容放入批处理文件中

		set CEF_VCVARS=none
		set GYP_MSVS_OVERRIDE_PATH=%VS2013%
		set CEF_USE_GN=0
		set DEPOT_TOOLS_WIN_TOOLCHAIN=0
		set GYP_DEFINES=buildtype=Official branding=Chromium proprietary_codecs=1 ffmpeg_branding=Chrome windows_sdk_path="C:\Program Files (x86)\Microsoft Visual Studio 12.0"
		set GYP_GENERATORS=ninja
		set GYP_MSVS_VERSION=2013
		set PATH=%WINSDK10%\bin\10.0.16299.0\x86;%VS2013%\VC\bin;%PATH%
		set LIB=%WINSDK10%\Lib\10.0.16299.0\um\x86;%WINSDK10%\Lib\10.0.16299.0\ucrt\x86;%VS2013%\VC\lib;%VS2013%\VC\atlmfc\lib;%LIB%
		set INCLUDE=%WINSDK10%\Include\10.0.16299.0\um;%WINSDK10%\Include\10.0.16299.0\ucrt;%WINSDK10%\Include\10.0.16299.0\shared;%WINSDK10%\Include\10.0.16299.0\winrt;%VS2013%\VC\include;%VS2013%\VC\atlmfc\include;%INCLUDE%
		call cef_create_projects.bat

9. 配置CEF工程
	cd chromium/src/cef
	./create_projects.bat

10. 更新libcef_dll_wrapper.ninja文件
	修改'chromium/src/out/Release/obj/cef/libcef_dll_wrapper.ninja'文件，把cflags中的'/MT'修改为'/MD'
	修改'chromium/src/out/Debug/obj/cef/libcef_dll_wrapper.ninja'文件，把cflags中的'/MTd'修改为'/MDd'

11. 编译工程
	cd chromium\src
	ninja libcef_dll_wrapper -C out\Release
	ninja libcef_dll_wrapper -C out\Debug
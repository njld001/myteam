##基于python的selenium自动化测试工具安装

###安装python-2.7.6.msi（默认安装即可）
####环境变量追加：
	C:\python27\

####安装setuptools-0.6c11.win32-py2.7.exe（默认安装即可）

####解压selenium-2.22.1.tar到C:\Python27\Lib\site-packages

	cmd
	C:
	cd C:\Python27\Lib\site-packages\selenium-2.40.0
	python setup.py install

####chromedriver_win32解压放到C:\Python27

	cd C:\Python27
	python ez_setup.py install

####将chrome安装路径的安装路径添加到环境变量中


###双击test.py文件能自动打开chrome浏览器显示百度一面就表示安装完成


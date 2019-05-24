PHP基础
1.	extension和zend_extension区别
extension 是基于php引擎的扩展（全路径/相对extension_dir路径）
zend_extension是基于zend引擎的扩展 （全路径）
如 opcache
zend_extension             non zts, non debug build
zend_extension_ts          zts,non debug build
zend_extension_debug       non zts, debug build
zend_extension_debug_ts    zts, debug build
zts(zend thread safety)
可以通过phpinfo中的zts值决定使用哪个


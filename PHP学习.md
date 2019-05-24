PHP基础
1. extension和zend_extension区别
extension 是基于php引擎的扩展（全路径/相对extension_dir路径）
zend_extension是基于zend引擎的扩展 （全路径）
如 opcache
zend_extension             non zts, non debug build
zend_extension_ts          zts,non debug build
zend_extension_debug       non zts, debug build
zend_extension_debug_ts    zts, debug build
zts(zend thread safety)
可以通过phpinfo中的zts值决定使用哪个
2. var_export和var_dump区别
var_export 
输出合法的php代码；可以传递第二个参数，从而返回结果；资源类型返回null
var_dump 
输出结构，展示值类型；资源类型输出resource；可以用控制输出函数捕获输出ob_get_contents()


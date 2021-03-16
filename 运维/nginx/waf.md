 

|          | modsecutity                                                  | openresty+nginx_lua_waf                                      |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **简介** | nginx官方plus版本中使用modsecurity作为waf，基于c，功能强大，但配置规则相对复杂 | 基于lua,需要nginx_lua模块，较为轻量，相对性能较好            |
| 功能     | ◆HTTP Protection （HTTP防御） - HTTP协议和本地定义使用的detectsviolations策略。<br>◆Real-time Blacklist Lookups（实时黑名单查询） -利用第三方IP信誉。<br/>◆HTTP Denial of Service Protections（HTTP的拒绝服务保护） -防御HTTP的洪水攻击和HTTP Dos 攻击。<br/>◆Common Web Attacks Protection（常见的Web攻击防护） -检测常见的Web应用程序的安全攻击。<br/>◆Automation Detection（自动化检测） -检测机器人，爬虫，扫描仪和其他表面恶意活动。<br/>◆Integration with AV Scanning for File Uploads（文件上传防病毒扫描） -检测通过Web应用程序上传的恶意文件。<br/>◆Tracking Sensitive Data（跟踪敏感数据） -信用卡通道的使用，并阻止泄漏。<br/>◆Trojan Protection（木马防护） -检测访问木马。<br/>◆Identification of Application Defects （应用程序缺陷的鉴定）-应用程序的错误配置警报。<br/>◆Error Detection and Hiding（错误检测和隐藏） -伪装服务器发送错误消息。 | 防止sql注入，本地包含，部分溢出，fuzzing测试，xss，SSRF等web攻击<br/>防止svn/备份之类文件泄露<br/>防止ApacheBench之类的压力测试工具的攻击<br/>屏蔽常见的黑客扫描工具，扫描器<br/>屏蔽异常的网络请求<br/>屏蔽图片附件类目录PHP执行权限<br/>防止webshell上传 |
| 优势     | waf规则多样，可定制性强                                      | 性能更优秀<br>配置快速，nginx动态编译两个模块                |
| 缺陷     | 定制规则较为麻烦                                             | 因为基于lua,所以需要nginx-lua插件模块                        |
| 现有规则 | OWASP Core Rule Set                                          | nginx_lua_waf                                                |

